# Running n8n in Docker (WSL2-Friendly)

This guide shows how to run **n8n**, the workflow automation tool, in Docker with persistent data and proper configuration for WSL2 environments.

Reference: [n8n Docker Docs](https://docs.n8n.io/hosting/installation/docker/)

---

## 1. Install Docker

Make sure Docker is installed and running. Check the version:

```bash
docker --version
````

---

## 2. Pull the n8n Docker Image

Pull the official n8n image:

```bash
docker pull n8nio/n8n
```

* Downloads the latest stable n8n image from Docker Hub.

---

## 3. Create a Docker Volume for Persistent Data

WSL2 can have permission issues with host-mounted folders. Use a Docker-managed volume:

```bash
docker volume create n8n_data
```

* `n8n_data` stores workflow data, credentials, and configuration safely.

---

## 4. Run n8n in Docker (Detached, with Authentication and Timezone)

```bash
docker run -d \
  --name n8n \
  -p 5678:5678 \
  -e N8N_HOST=0.0.0.0 \
  -e GENERIC_TIMEZONE="America/New_York" \
  -e TZ="America/New_York" \
  -e N8N_BASIC_AUTH_ACTIVE=true \
  -e N8N_BASIC_AUTH_USER=admin \
  -e N8N_BASIC_AUTH_PASSWORD=supersecret \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n
```

**Explanation of each option:**

* `-d` → Run container in background (detached mode).
* `--name n8n` → Names the container `n8n`.
* `-p 5678:5678` → Maps container port 5678 to host port 5678.
* `-e N8N_HOST=0.0.0.0` → Ensures n8n listens on all interfaces (needed for WSL2).
* `-e GENERIC_TIMEZONE` and `-e TZ` → Sets timezone for scheduled workflows.
* `-e N8N_BASIC_AUTH_ACTIVE` → Enables basic auth.
* `-e N8N_BASIC_AUTH_USER` and `-e N8N_BASIC_AUTH_PASSWORD` → Authentication credentials.
* `-v n8n_data:/home/node/.n8n` → Mounts persistent Docker volume for data.

---

## 5. Check Logs

If the container exits unexpectedly:

```bash
docker logs -f n8n
```

Expected output:

```
Server started on 0.0.0.0:5678
```

---

## 6. Access n8n

Open a browser in Windows:

```
http://localhost:5678
```

Log in using the username/password set in the environment variables (`admin` / `supersecret`).

---

## 7. Activate Community Edition License

1. Fill out the activation form.
2. Check your email for the license key.
3. Open:

```
http://localhost:5678/settings/usage
```

4. Paste your license key to activate the community edition.

---

## 8. Stop, Remove, and Cleanup

### Stop the container:

```bash
docker stop n8n
```

### Remove the container:

```bash
docker rm n8n
```

### Remove the Docker image (if needed):

```bash
docker rmi n8nio/n8n
```

### Remove the Docker volume (if needed):

```bash
docker volume rm n8n_data
```

> ⚠️ **Warning:** Removing the volume deletes all n8n data permanently.

---

### Notes

* Using a Docker volume avoids file permission issues common in WSL2.
* Environment variables like `N8N_HOST`, `TZ`, and `GENERIC_TIMEZONE` are required for proper operation.
* For long-term use, a `docker-compose.yml` setup is recommended.

```

