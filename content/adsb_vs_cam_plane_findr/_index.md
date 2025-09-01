---
title: 1 - Project Outline
type: docs
prev: adsb_vs_cam_plane_findr/
next: adsb_vs_cam_plane_findr/
sidebar:
  open: true
math: true
---

## Project Requiremnts

1) listen to adsb data
 	 - geofence event triggered based on adsb data
	 - geofence defined in kml files and created with using polygons in google earth pro

2) start cameras taking pictures
   - take pictures from multiple cameras
   - save images to disk with timestamp and camera id
   - cameras triggered by geofence event

3) use yolo to detect airplane in image
   - find aircraft
	  - determine pixel coordinates of aircraft in image
   - identify difference between frames to identify motion?

4) 3d triangulation using contour map is undeveloped
   - take pixel coordinates and convert into 3d rays using CV2
   - get camera intrinsics and extrinsics from opencv calibration
   - get vector going from camera to the airplane
   - if can get two or more 3d rays from different cameras, then can triangulate plane
	 - skew lines, find where they are smallest

5) then need to compare triangulated position to real time adsb data
   - lat long altitude
   - UTM coordinates

6) need to connect webapp to backend
   - make set of alerts
     - adsb geofence trigger
     - Yolo detects aircraft
     - aircraft adsb location vs triangulation position error out of tolerance (ex 10 meters)
     - quality of service metric
     - camera online/offline
     - camera health (temp, cpu load, disk space, etc) ?
     - server health (temp, cpu load, disk space, etc) ?
     - NAS health (temp, cpu load, disk space, etc) ?


Run geofence system separately from triangulation system?
Run calculations in parallel process?

Training a new model will happen in the future but stick to the basics for now.

## Tech Outline

> starting with a PoC then trying to dockerize everything

* Event-Driven Workflow
* services Architecture
* Message Broker: RabbitMQ
* FastAPI for Backend API

**Plausible Directory Structure**

```
project-root/
│── docker-compose.yml        # Orchestrates RabbitMQ + services
│── .env                      # Env vars (Rabbit host, credentials, etc.)
│
├── services/                 # All worker scripts
│   ├── adsb_geofence.py
│   ├── cameras_take_pictures.py
│   ├── yolo_plane_finder.py
│   ├── plane_triangulation.py
│   ├── adsb_vs_triangulation.py
│   ├── hardware_health.py
│   └── __init__.py
│
├── api/                      # FastAPI backend
│   ├── fastapi_service.py
│   ├── __init__.py
│
├── configs/                  # Configs & static assets
│   ├── rabbitmq.conf
│   ├── cameras.yaml
│   ├── geofence.kml
│   └── health_config.yaml    # thresholds for alerts
│
├── requirements.txt          # Shared dependencies
└── Dockerfiles/              # Each service can have its own Dockerfile if needed
    ├── Dockerfile.adsb
    ├── Dockerfile.cameras
    ├── Dockerfile.yolo
    ├── Dockerfile.triangulation
    ├── Dockerfile.adsbtri
    ├── Dockerfile.health
    ├── Dockerfile.api

```

**Service Responsibilities**

1. **ADSB Geofence Service (`adsb_geofence.py`)**
   - Connects to live ADS-B feed (network or SDR dongle).
   - Loads geofence polygons from KML files.
   - Checks if any aircraft enter/exit geofenced areas defined in KML files.
   - Publishes geofence events to RabbitMQ.

2. **Camera Control Service (`cameras_take_pictures.py`)**
   - Subscribes to geofence_triggered events.
   - Starts/stops cameras when a geofence event occurs.
   - Captures images from multiple cameras.
   - Saves images to disk with metadata (timestamp, camera ID).
   - Publishes “new_frame” messages with metadata (timestamp, camera ID, file path).

3. **YOLO Plane Finder Service (`yolo_plane_finder.py`)**
   - Subscribes to “new_frame” messages.
   - Loads YOLO model (pre-trained or custom).
   - Processes images to detect airplanes.
   - Extracts pixel coordinates of detected airplanes.
   - Publishes “plane_detected” messages with pixel coordinates and metadata.

4. **Plane Triangulation Service (`plane_triangulation.py`)**
   - Loads camera calibration data (intrinsics, extrinsics).
   - Subscribes to “plane_detected” messages from multiple cameras.
   - uses OpenCV extrinsics and intrinsics to convert pixel coordinates to 3D rays.
   - Performs triangulation to estimate 3D position of the airplane.
     - Handles skew lines and finds the closest point between rays.
     - Ideally >= 2 cameras needed for triangulation.
   - Publishes “triangulated_position” messages with 3D coordinates and metadata.
   - Handles cases with insufficient data (e.g., only one camera detects the plane).

5. **ADSB vs Triangulation Comparison Service (`adsb_vs_triangulation.py`)**
    - Subscribes to “triangulated_position” messages.
    - Retrieves corresponding ADS-B data for the same timestamp.
    - Compares triangulated position with ADS-B reported position.
    - Calculates error metrics (distance between triangulated and ADS-B positions).
    - Publishes alerts if error exceeds predefined thresholds.

6. **Hardware Health Monitoring Service (`hardware_health.py`)**
    - Monitors system health metrics (CPU, memory, disk space, temperature).
    - Publishes health status messages at regular intervals.
    - Alerts if any metric exceeds predefined thresholds.

7. **RabbitMQ Message Broker**
    - your message backbone (RabbitMQ broker).
    - All other services connect here to publish/subscribe to messages.
    - Local dev: one RabbitMQ Docker container.
    - Later: clustered RabbitMQ in Kubernetes

8. **FastAPI Backend Service (`fastapi_service.py`)**
   - Provides REST/WebSocket API to the outside world.
   - Subscribes to alerts + system health events.
   - Persists results into a DB/Redis if needed.
   - Endpoints might include:
     - /alerts → recent alerts (geofence, mismatches, YOLO detections)
       - /alerts/time-range → be able to find old alerts or return all alerts from a time delta with dates 
     - /system_metrics → system health (cameras, workers, server stats, uptime)
     - /aircraft/{icao} → current ADS-B vs triangulated position
     - /media/{camera_id}/{timestamp} → retrieve images/videos
     - /weather → current weather data from API
     - /config → view/update system configs (geofences, thresholds)
     - /settings → view/update camera settings
     - /docs → auto-generated API docs (Swagger UI)


**Dockerization & Deployment**
- Each service has its own Dockerfile.
- Use Docker Compose for local development (RabbitMQ + all services).
- Use environment variables for configuration (RabbitMQ host, credentials, etc.).
- Volumes for persistent storage (images, logs).

Plausible docker Compose example:

```yaml
version: "3.9"

services:
  rabbitmq:
    image: rabbitmq:3-management
    container_name: rabbitmq
    ports:
      - "5672:5672"      # AMQP
      - "15672:15672"    # Management UI
    environment:
      RABBITMQ_DEFAULT_USER: user
      RABBITMQ_DEFAULT_PASS: pass
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

  fastapi:
    build:
      context: .
      dockerfile: Dockerfiles/Dockerfile.api
    container_name: fastapi_service
    depends_on:
      - rabbitmq
    environment:
      - RABBITMQ_HOST=rabbitmq
    ports:
      - "8000:8000"

  adsb_geofence:
    build:
      context: .
      dockerfile: Dockerfiles/Dockerfile.adsb
    depends_on:
      - rabbitmq
    environment:
      - RABBITMQ_HOST=rabbitmq

  cameras:
    build:
      context: .
      dockerfile: Dockerfiles/Dockerfile.cameras
    depends_on:
      - rabbitmq
    environment:
      - RABBITMQ_HOST=rabbitmq

  yolo:
    build:
      context: .
      dockerfile: Dockerfiles/Dockerfile.yolo
    depends_on:
      - rabbitmq
    environment:
      - RABBITMQ_HOST=rabbitmq

  triangulation:
    build:
      context: .
      dockerfile: Dockerfiles/Dockerfile.triangulation
    depends_on:
      - rabbitmq
    environment:
      - RABBITMQ_HOST=rabbitmq

  adsb_vs_triangulation:
    build:
      context: .
      dockerfile: Dockerfiles/Dockerfile.adsbtri
    depends_on:
      - rabbitmq
    environment:
      - RABBITMQ_HOST=rabbitmq

  hardware_health:
    build:
      context: .
      dockerfile: Dockerfiles/Dockerfile.health
    depends_on:
      - rabbitmq
    environment:
      - RABBITMQ_HOST=rabbitmq

volumes:
  rabbitmq_data:
```