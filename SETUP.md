# AgentCenter — Self-Hosted Setup Guide

Run AgentCenter on your own server using Docker.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Quick Start](#quick-start)
3. [Step-by-Step Setup](#step-by-step-setup)
4. [HTTPS / Reverse Proxy](#https--reverse-proxy)
5. [Optional Features](#optional-features)
6. [Upgrading](#upgrading)
7. [Backups & Restore](#backups--restore)
8. [Maintenance](#maintenance)
9. [Troubleshooting](#troubleshooting)
10. [Environment Variables Reference](#environment-variables-reference)

---

## Prerequisites

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| Docker | v20.10+ | Latest stable |
| Docker Compose | v2.0+ | Latest (bundled with Docker Desktop) |
| RAM | 2 GB | 4 GB+ |
| Disk | 10 GB | 20 GB+ |
| OS | Any Docker-compatible OS | Ubuntu 22.04+ / Debian 12+ |
| Network | Outbound HTTPS (port 443) | Static IP or domain for HTTPS |

You also need a **license key** — purchase one at [agentcenter.cloud](https://agentcenter.cloud).

### Verify Docker is installed

```bash
docker --version       # Docker version 20.10+
docker compose version # Docker Compose v2+
```

If Docker is not installed:

```bash
# Ubuntu / Debian
curl -fsSL https://get.docker.com | sh
sudo systemctl enable docker && sudo systemctl start docker

# Add your user to the docker group (avoids needing sudo)
sudo usermod -aG docker $USER
# Log out and back in for the group change to take effect
```

---

## Quick Start

```bash
mkdir agentcenter && cd agentcenter
# Copy docker-compose.yml and .env.example into this directory

cp .env.example .env
sed -i "s|^PG_PASS=.*|PG_PASS=$(openssl rand -hex 16)|" .env
sed -i "s|^REDIS_PASS=.*|REDIS_PASS=$(openssl rand -hex 16)|" .env
sed -i "s|^AUTH_SECRET=.*|AUTH_SECRET=$(openssl rand -hex 32)|" .env
sed -i "s|^CRON_SECRET=.*|CRON_SECRET=$(openssl rand -hex 32)|" .env

# Edit PUBLIC_URL to match your domain (skip if testing locally)
# nano .env

docker compose up -d
```

Open `http://localhost:3000/activate`, activate your license, and create your admin account.

> **macOS users:** Replace `sed -i` with `sed -i ''` (BSD sed) in the commands above.

---

## Step-by-Step Setup

### Step 1: Create the project directory

```bash
mkdir agentcenter && cd agentcenter
```

### Step 2: Place the configuration files

Copy the provided files into your project directory:
- `docker-compose.yml` — defines the services (PostgreSQL, Redis, AgentCenter app)
- `.env.example` — template with all configuration options
- `docker-compose.prod.yml` — Traefik HTTPS overlay (optional, for production)

### Step 3: Configure environment variables

```bash
cp .env.example .env
```

Set the **required** values in `.env`:

```bash
PG_PASS=<run: openssl rand -hex 16>
REDIS_PASS=<run: openssl rand -hex 16>
AUTH_SECRET=<run: openssl rand -hex 32>
CRON_SECRET=<run: openssl rand -hex 32>

# The URL your users will access the app at
PUBLIC_URL=https://agents.yourcompany.com
```

> **Tip:** Generate all secrets at once:
> ```bash
> echo "PG_PASS=$(openssl rand -hex 16)"
> echo "REDIS_PASS=$(openssl rand -hex 16)"
> echo "AUTH_SECRET=$(openssl rand -hex 32)"
> echo "CRON_SECRET=$(openssl rand -hex 32)"
> ```

### Step 4: Start the services

```bash
docker compose up -d
```

Verify everything is running:

```bash
docker compose ps
```

You should see three containers with status `Up` (or `Up (healthy)`):

```
NAME                    STATUS
agentcenter-app-1       Up
agentcenter-postgres-1  Up (healthy)
agentcenter-redis-1     Up (healthy)
```

### Step 5: Activate your license

1. Open `http://localhost:3000/activate` (or your `PUBLIC_URL/activate`)
2. Paste the **license key** you received after purchasing from developer
3. Click **Activate**

### Step 6: Create your admin account

After activation, create your admin account — this is the first user with full dashboard access.

### Step 7: Done

Open the dashboard and start creating agents and tasks.

---

## HTTPS / Reverse Proxy

For production, you **must** use HTTPS.

### Before you start

You need:

1. **A domain name** — e.g., `agents.yourcompany.com`
2. **A public IP address** — your server must be reachable from the internet
3. **DNS A record** pointing your domain to the server IP:

   ```
   Type: A
   Name: agents          (or your subdomain)
   Value: 203.0.113.10   (your server's public IP)
   TTL: 300
   ```

4. **Ports 80 and 443 open** in your firewall/security group

Find your server's public IP:

```bash
curl -s ifconfig.me
```

Verify DNS:

```bash
dig agents.yourcompany.com +short
# Should return your server's public IP
```

> **Note:** SSL certificates cannot be issued for bare IP addresses — you need a domain.

### Option A: Caddy (recommended — automatic HTTPS)

Install Caddy ([docs](https://caddyserver.com/docs/install)), then create `/etc/caddy/Caddyfile`:

```
agents.yourcompany.com {
    reverse_proxy localhost:3000
}
```

```bash
sudo systemctl enable caddy && sudo systemctl start caddy
```

### Option B: nginx + Let's Encrypt

```bash
sudo apt install nginx certbot python3-certbot-nginx -y
```

Create `/etc/nginx/sites-available/agentcenter`:

```nginx
server {
    listen 80;
    server_name agents.yourcompany.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/agentcenter /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
sudo certbot --nginx -d agents.yourcompany.com
```

### Option C: Traefik (via Docker — built-in)

A `docker-compose.prod.yml` overlay is included. It adds Traefik with automatic HTTPS.

1. Add to your `.env`:

```bash
DOMAIN=agents.yourcompany.com
ACME_EMAIL=admin@yourcompany.com
```

2. Start with both compose files:

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### Update PUBLIC_URL

After setting up HTTPS, make sure `.env` has:

```bash
PUBLIC_URL=https://agents.yourcompany.com
```

Then restart: `docker compose up -d app`

---

## Optional Features

### File Uploads (Cloudflare R2)

Add to `.env`:

```bash
R2_ACCOUNT_ID=your-account-id
R2_ACCESS_KEY_ID=your-access-key
R2_SECRET_ACCESS_KEY=your-secret-key
R2_BUCKET_NAME=your-bucket-name
```

Restart: `docker compose up -d app`

### Push Notifications (VAPID)

Generate keys: `npx web-push generate-vapid-keys`

Add to `.env`:

```bash
VAPID_PUBLIC_KEY=BPxr...
VAPID_PRIVATE_KEY=dGhp...
VAPID_SUBJECT=mailto:admin@yourcompany.com
```

Restart: `docker compose up -d app`

### Email Notifications (Resend)

Sign up at [resend.com](https://resend.com), then add to `.env`:

```bash
RESEND_API_KEY=re_...
NOTIFICATION_FROM_EMAIL=notifications@yourcompany.com
```

Restart: `docker compose up -d app`

### Custom Port

```bash
# In .env
PORT=8080
```

---

## Upgrading

```bash
cd agentcenter

# Pull the latest image
docker compose pull

# Restart — migrations run automatically
docker compose up -d
```

### Pin a specific version

```bash
# In .env
APP_VERSION=v1.2.3

docker compose up -d
```

---

## Backups & Restore

### Database backup

```bash
docker compose exec postgres pg_dump -U postgres agentcenter > backup-$(date +%Y%m%d).sql
```

### Restore from backup

```bash
docker compose exec -T postgres psql -U postgres agentcenter < backup.sql
```

### Automated daily backups

```bash
crontab -e
```

Add:

```
0 2 * * * cd /path/to/agentcenter && docker compose exec -T postgres pg_dump -U postgres agentcenter | gzip > /path/to/backups/agentcenter-$(date +\%Y\%m\%d).sql.gz
```

---

## Maintenance

### View logs

```bash
docker compose logs -f app          # App logs
docker compose logs --tail 100 app  # Last 100 lines
```

### Restart a service

```bash
docker compose restart app
docker compose restart postgres
docker compose restart redis
```

### Check disk usage

```bash
docker system df
docker compose exec postgres psql -U postgres -c "SELECT pg_size_pretty(pg_database_size('agentcenter'));"
```

### Clean up old images

```bash
docker image prune -f
```

---

## Troubleshooting

### Container won't start

```bash
docker compose ps
docker compose logs app
sudo lsof -i :3000   # Check if port is in use
```

### Database connection errors

```bash
docker compose exec postgres pg_isready -U postgres
docker compose exec postgres psql -U postgres -d agentcenter -c "SELECT 1;"
```

### "502 Bad Gateway" behind reverse proxy

- Check the app is running: `docker compose ps`
- Check the app responds: `curl -s http://localhost:3000/api/health`
- Verify proxy points to the correct port

### License issues

| Error | Solution |
|-------|----------|
| "License expired" | Your subscription may have lapsed. Contact support. |
| "Already active on another instance" | Wait 2 hours for the old instance to expire, or contact support. |

### Reset everything (delete all data)

```bash
docker compose down -v   # Removes containers AND volumes
docker compose up -d     # Fresh start
```

> **Warning:** This permanently deletes all data. You will need to re-activate your license.

---

## Environment Variables Reference

### Required

| Variable | Description |
|----------|-------------|
| `PG_PASS` | PostgreSQL password. Generate: `openssl rand -hex 16` |
| `REDIS_PASS` | Redis password. Generate: `openssl rand -hex 16` |
| `AUTH_SECRET` | Session secret (min 16 chars). Generate: `openssl rand -hex 32` |
| `CRON_SECRET` | Internal API secret. Generate: `openssl rand -hex 32` |
| `PUBLIC_URL` | URL users access the app at (e.g., `https://agents.yourcompany.com`) |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_VERSION` | `latest` | Docker image tag |
| `PORT` | `3000` | Host port |
| `R2_ACCOUNT_ID` | — | Cloudflare R2 account ID |
| `R2_ACCESS_KEY_ID` | — | R2 access key |
| `R2_SECRET_ACCESS_KEY` | — | R2 secret key |
| `R2_BUCKET_NAME` | — | R2 bucket name |
| `VAPID_PUBLIC_KEY` | — | Web push public key |
| `VAPID_PRIVATE_KEY` | — | Web push private key |
| `VAPID_SUBJECT` | — | Web push subject (e.g., `mailto:admin@yourcompany.com`) |
| `RESEND_API_KEY` | — | Resend email API key |
| `NOTIFICATION_FROM_EMAIL` | — | Sender email address |

### Production HTTPS (Traefik)

| Variable | Description |
|----------|-------------|
| `DOMAIN` | Your domain (e.g., `agents.yourcompany.com`) |
| `ACME_EMAIL` | Email for Let's Encrypt certificate notifications |

---

## Getting Help

- Email: dharmik.jagodana@agentcenter.cloud
- Website: [agentcenter.cloud](https://agentcenter.cloud)