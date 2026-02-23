# Zammad Docker Deployment Guide (with Nginx Proxy Manager & SSL)

This guide walks you through setting up **Zammad** on a Hetzner cloud server using **Docker Compose**, with **HTTPS via Nginx Proxy Manager**, and support for email via SMTP/IMAP.

---

## Prerequisites

- Ubuntu 22.04 LTS server (Hetzner Cloud)
- Domain: `hg-hausverwalter.de`
- Subdomain: `zammad.hg-hausverwalter.de`
- Mailbox: `zammad@hg-hausverwalter.de`
- Docker & Docker Compose installed
- DNS A record for `zammad.hg-hausverwalter.de` pointing to server IP

---

## 1. Set up Directory and Clone Repository

```bash
cd /opt
git clone https://github.com/zammad/zammad-docker-compose.git
cd zammad-docker-compose
cp env.dist .env
```

---

## 2. Configure Environment (.env)

Edit `.env` file:

```bash
nano .env
```

Update the following:

```env
ZAMMAD_FQDN=zammad.hg-hausverwalter.de
ZAMMAD_HTTP_TYPE=https
RAILS_TRUSTED_PROXIES=0.0.0.0/0

VIRTUAL_HOST=zammad.hg-hausverwalter.de
VIRTUAL_PORT=8080
LETSENCRYPT_HOST=zammad.hg-hausverwalter.de
LETSENCRYPT_EMAIL=you@hg-hausverwalter.de
```

---

## 3. Enable Nginx Proxy Manager Scenario

```bash
cp scenarios/add-nginx-proxy-manager.yml docker-compose.override.yml
```

---

## 4. Pull Images and Start the Stack

```bash
docker compose pull
docker compose up -d
```

Wait for all containers to start.

---

## 5. Open Nginx Proxy Manager UI

Access: `http://<your-server-ip>:81`

Login:
- **Email**: `admin@example.com`
- **Password**: `changeme`

Set new admin email and password.

---

## 6. Configure Proxy Host for Zammad

- Go to **Proxy Hosts > Add Proxy Host**
- **Domain Name**: `zammad.hg-hausverwalter.de`
- **Forward Hostname/IP**: `zammad-nginx`
- **Forward Port**: `8080`
- **SSL Tab**:
  - Check "Request a new SSL Certificate"
  - Enable Websockets
  - Enable Force SSL, HTTP/2, and HSTS
  - Use your real email for Let's Encrypt
- Click **Save**

Zammad will now be accessible at:
```
https://zammad.hg-hausverwalter.de
```

---

## 7. Configure Email (SMTP/IMAP)

Log in to the Zammad admin interface and go to:

### Outgoing (SMTP):
- **Admin > Channels > Email > Outbound**
- Add `zammad@hg-hausverwalter.de`
- Configure SMTP:
  - SMTP Server: your provider's SMTP (e.g. smtp.yourmail.com)
  - Port: 587 (TLS) or 465 (SSL)
  - Auth enabled
  - Username: `zammad@hg-hausverwalter.de`
  - Password: (your email password)

### Incoming (IMAP - optional):
- **Admin > Channels > Email > Inbound**
- Add `zammad@hg-hausverwalter.de`
- Configure IMAP:
  - IMAP server and port
  - Username and password

---

## 8. Optional: Add DNS Records for Email Deliverability

In your DNS provider:

- **SPF**:
```txt
v=spf1 include:_spf.yourmailhost.com ~all
```

- **DKIM/DMARC**: Configure via your mail provider's instructions

---

## 9. Done!

Zammad is now fully installed with:
- Docker Compose
- SSL via Nginx Proxy Manager
- Custom domain & email

You can now start managing support tickets from your browser at:
```
https://zammad.hg-hausverwalter.de
```
