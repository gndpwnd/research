## Table of Contents

- [LocalTunnel Guide – Free Tunnels for Multiple Apps](#localtunnel-guide-–-free-tunnels-for-multiple-apps)
  - [1. Install Node.js Locally (No sudo)](#1.-install-node.js-locally-(no-sudo))
  - [2. Install LocalTunnel](#2.-install-localtunnel)
  - [3. Prepare a Log Directory](#3.-prepare-a-log-directory)
  - [4. Run Your Local Service](#4.-run-your-local-service)
  - [5. Expose Ports via LocalTunnel (Background + Logging)](#5.-expose-ports-via-localtunnel-(background-+-logging))
  - [6. Get Tunnel Password (Optional for Visitors)](#6.-get-tunnel-password-(optional-for-visitors))
  - [7. Optional: Custom Subdomain](#7.-optional:-custom-subdomain)
  - [8. Notes & Caveats](#8.-notes-&-caveats)

# LocalTunnel Guide – Free Tunnels for Multiple Apps

LocalTunnel is a free, simple way to expose your local apps to the internet. You can run multiple tunnels for different ports and keep them running in the background with logs.

---

## 1. Install Node.js Locally (No sudo)

If Node.js is not installed system-wide, install it locally:

```bash
# Create a local npm folder
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'

# Add npm binaries to PATH
export PATH="$HOME/.npm-global/bin:$PATH"

# Make PATH permanent
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
````

Check installation:

```bash
node -v
npm -v
```

---

## 2. Install LocalTunnel

```bash
npm install -g localtunnel
```

This installs `lt` in your home folder, no sudo required.

---

## 3. Prepare a Log Directory

```bash
mkdir -p ~/logs/localtunnel/
```

All tunnels will log here.

---

## 4. Run Your Local Service

Examples:

**Flask App:**

```bash
python app.py
# Default port: 5000
```

**Docker Container:**

```bash
docker run -p 5000:5000 myapp
```

---

## 5. Expose Ports via LocalTunnel (Background + Logging)

**Single port example (5000):**

```bash
nohup lt --port 5000 > ~/logs/localtunnel/lt_5000.log 2>&1 &
```

**Multiple ports example:**

```bash
nohup lt --port 5000 > ~/logs/localtunnel/lt_5000.log 2>&1 &
nohup lt --port 8000 > ~/logs/localtunnel/lt_8000.log 2>&1 &
nohup lt --port 8080 > ~/logs/localtunnel/lt_8080.log 2>&1 &
```

**Check running tunnels:**

```bash
ps aux | grep lt
tail -f ~/logs/localtunnel/lt_5000.log
```

---

## 6. Get Tunnel Password (Optional for Visitors)

LocalTunnel free tunnels show a password page for some visitors. It is the ip address of the hosting machine. Get your tunnel password:

```bash
curl https://loca.lt/mytunnelpassword
```

Share this password with your users if needed.

---

## 7. Optional: Custom Subdomain

You can request a subdomain (if available):

```bash
lt --port 5000 --subdomain myapp > ~/logs/localtunnel/lt_5000.log 2>&1 &
```

---

## 8. Notes & Caveats

* Each tunnel URL is **public**, anyone who guesses it can access your service. Use authentication if necessary.
* Free tunnels are **ephemeral** and may change after restarting.
* Tunnel speed and reliability may be lower than paid services like ngrok.
* Multiple tunnels on different ports can coexist without interfering with each other.