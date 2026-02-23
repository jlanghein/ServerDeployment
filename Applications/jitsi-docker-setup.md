# Self-Hosted Jitsi with Docker, Caddy, and Auth

This guide walks you through setting up a secure, self-hosted instance of [Jitsi Meet](https://jitsi.org/) on a Hetzner Cloud server using Docker Compose, with HTTPS via Caddy and user authentication enabled.

---

## Prerequisites

- A Hetzner cloud server (tested on CPX21)
- Docker & Docker Compose installed
- A public domain (e.g., `atlanhomeinc.com`)
- `meet.atlanhomeinc.com` DNS A record pointing to your server
- Caddy reverse proxy running (e.g., via `lucaslorentz/caddy-docker-proxy`)
- Seafile is already running on the same server using Docker and `seafile-net`

---

## Directory Structure

```bash
/opt/jitsi/
  ├── docker-compose.yml
  └── .env
```

---

## Step 1: Clone and Prepare

```bash
cd /opt
git clone https://github.com/jitsi/docker-jitsi-meet jitsi
cd jitsi
cp env.example .env
./gen-passwords.sh
```

Edit `.env` and set:

```env
PUBLIC_URL=https://meet.atlanhomeinc.com
DOCKER_HOST_ADDRESS=YOUR_SERVER_PUBLIC_IP
ENABLE_AUTH=1
ENABLE_GUESTS=1
AUTH_TYPE=internal
ENABLE_XMPP_WEBSOCKET=1
ENABLE_COLIBRI_WEBSOCKET=1
ENABLE_HTTPS=0
```

Replace `YOUR_SERVER_PUBLIC_IP` with your actual IP.

---

## Step 2: Configure Docker Compose

In `docker-compose.yml`:

### Web container

- Bind to localhost
- Add Caddy labels
- Join `seafile-net`

```yaml
  web:
    image: jitsi/web:${JITSI_IMAGE_VERSION:-unstable}
    restart: ${RESTART_POLICY:-unless-stopped}
    ports:
      - "127.0.0.1:8443:80"
    volumes:
      - ${CONFIG}/web:/config:Z
      ...
    labels:
      caddy: meet.atlanhomeinc.com
      caddy.reverse_proxy: "{{upstreams 80}}"
    networks:
      meet.jitsi:
      seafile-net:
```

### JVB (for media/WebSockets)

Expose Colibri WebSocket:

```yaml
  jvb:
    ...
    ports:
      - '${JVB_PORT:-10000}:${JVB_PORT:-10000}/udp'
      - '127.0.0.1:9090:9090'
```

### Add `seafile-net` at the bottom

```yaml
networks:
  meet.jitsi:
  seafile-net:
    external: true
```

---

## Step 3: Add DNS Record

In your DNS provider:

- Create an `A` record:  
  **Name**: `meet`  
  **Type**: `A`  
  **Value**: `YOUR_SERVER_IP`

---

## Step 4: Start the Stack

```bash
docker compose down
docker compose up -d
docker restart seafile-caddy
```

Caddy will auto-discover the container and provide HTTPS via Let's Encrypt.

---

## Step 5: Create Admin User

```bash
docker exec -it jitsi-prosody-1 prosodyctl --config /config/prosody.cfg.lua register johannes meet.jitsi yourpassword
```

Repeat for additional users if needed.

---

## User Behavior

- Only registered users can **create** or **start** a room.
- Guests can **join** after the room is started.
- When all users leave, the room is destroyed.
- Guests cannot rejoin unless a registered user re-starts the room.

---

## Firewall (optional)

If using UFW:

```bash
sudo ufw allow 10000/udp
```

---

## Tips

- Use `docker logs` to monitor JVB, Jicofo, or Prosody
- Backup config and user data with:
  ```bash
  tar czf jitsi-backup-$(date +%F).tar.gz /opt/jitsi/.jitsi-meet-cfg
  ```
- To enforce login for everyone, set `ENABLE_GUESTS=0`

---

Enjoy your secure, self-hosted Jitsi Meet!
