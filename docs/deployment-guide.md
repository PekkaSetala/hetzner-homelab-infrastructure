# Deployment Guide

Full setup from zero to running n8n on a Hetzner VPS with Coolify and Cloudflare.

## Prerequisites

- **Hetzner Cloud account** — [hetzner.com](https://www.hetzner.com/)
- **Domain name** — any registrar
- **Cloudflare account** — free tier
- **SSH client** and basic Linux/DNS knowledge

## Step 1: Provision Hetzner Server

1. Hetzner Cloud Console → Add Server
2. Location: e.g. Nuremberg
3. Image: **Ubuntu 24.04**
4. Type: **CX22** (4 GB RAM, 2 vCPU) — ~€5.83/month
5. Add your SSH key
6. Create

```bash
ssh root@YOUR_SERVER_IP

apt update && apt upgrade -y
timedatectl set-timezone Europe/Helsinki

# Firewall
apt install ufw -y
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 443/udp
ufw enable
```

## Step 2: Install Coolify

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash
```

Takes 3–5 minutes. Installs Docker, Traefik, PostgreSQL, Redis, and the Coolify app.

1. Open `http://YOUR_SERVER_IP:8000`
2. Create admin account
3. Verify server health under Servers → localhost

## Step 3: Configure Cloudflare DNS

1. Add your domain to Cloudflare, update nameservers at your registrar
2. Create A records (proxied):

| Type | Name | Content |
|------|------|---------|
| A | n8n | YOUR_SERVER_IP |
| A | app | YOUR_SERVER_IP |

3. SSL/TLS → set to **Full (strict)**
4. Enable: Always Use HTTPS, Automatic HTTPS Rewrites

See [Cloudflare Integration](cloudflare-integration.md) for details.

## Step 4: Deploy n8n

1. Coolify → Projects → New Project → "AI Automation"
2. \+ New Resource → Service → n8n
3. Set domain: `n8n.example.com`, enable SSL
4. Environment variables:
   ```
   GENERIC_TIMEZONE=Europe/Helsinki
   N8N_EDITOR_BASE_URL=https://n8n.example.com
   WEBHOOK_URL=https://n8n.example.com
   ```
5. Deploy — wait for health checks (~2 min)
6. Access at `https://n8n.example.com`, create admin account

Verify:
```bash
docker ps | grep n8n
docker logs n8n-{ID} --tail 50
```

## Step 5: Configure Coolify Dashboard Domain

1. Coolify Settings → Configuration → set domain to `app.example.com`
2. Traefik picks up the change automatically

## Step 6: Verify

| Service | URL | Expected |
|---------|-----|----------|
| n8n | https://n8n.example.com | Login page |
| Coolify | https://app.example.com:8000 | Dashboard |

```bash
# Check all containers are healthy
docker ps --format "table {{.Names}}\t{{.Status}}"

# Verify TLS
curl -vI https://n8n.example.com 2>&1 | grep "subject:"
```

## Step 7: Backups

Set up automated backups using the included scripts:

```bash
# Test backup
./scripts/backup-procedures.sh

# Schedule daily at 2 AM
crontab -e
0 2 * * * /root/scripts/backup-procedures.sh
```

See [`scripts/backup-procedures.sh`](../scripts/backup-procedures.sh) and [`scripts/restore-procedures.sh`](../scripts/restore-procedures.sh) for details.

## Estimated Cost

| Item | Monthly |
|------|---------|
| Hetzner CX22 | €5.83 |
| Domain | €1–2 |
| Cloudflare | Free |
| **Total** | **~€7–8** |
