# FusionPBX HTTPS Setup Guide (Let's Encrypt + NGINX)

This guide walks you through securing your FusionPBX installation with a free Let's Encrypt SSL certificate using Certbot and NGINX.

---

## Prerequisites

- Domain name (e.g., `fusion.atlanhomeinc.com`) pointed to your server's IP
- NGINX installed and serving FusionPBX
- Root or sudo access

---

## Step-by-Step Instructions

### 1. Install Certbot & NGINX Plugin

**Debian/Ubuntu:**
```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

**CentOS/RHEL:**
```bash
sudo yum install epel-release -y
sudo yum install certbot python3-certbot-nginx -y
```

---

### 2. Ensure Domain is Reachable

```bash
ping fusion.atlanhomeinc.com
```

---

### 3. Request SSL Certificate

```bash
sudo certbot --nginx -d fusion.atlanhomeinc.com
```

- Enter your email
- Agree to terms
- Choose to redirect HTTP to HTTPS

---

### 4. Fix NGINX Config (If Needed)

If Certbot fails with a "Could not automatically find a matching server block" error:

#### Create or edit NGINX config:
```bash
sudo nano /etc/nginx/sites-available/fusion.atlanhomeinc.com
```

#### Paste this config (adjust PHP version if needed):
```nginx
server {
    listen 80;
    server_name fusion.atlanhomeinc.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name fusion.atlanhomeinc.com;

    ssl_certificate /etc/letsencrypt/live/fusion.atlanhomeinc.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/fusion.atlanhomeinc.com/privkey.pem;

    root /var/www/fusionpbx;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

#### Enable the site (if using sites-available):
```bash
sudo ln -s /etc/nginx/sites-available/fusion.atlanhomeinc.com /etc/nginx/sites-enabled/
```

---

### 5. Reload NGINX

```bash
sudo nginx -t && sudo systemctl reload nginx
```

---

### 6. Test in Browser

Visit: https://fusion.atlanhomeinc.com

You should see the secure FusionPBX login screen.

---

### 7. Verify Auto-Renewal

```bash
sudo certbot renew --dry-run
```

This ensures your certificate will auto-renew every 90 days.

---

## Done!

Your FusionPBX instance is now secured with HTTPS and ready for production use.
