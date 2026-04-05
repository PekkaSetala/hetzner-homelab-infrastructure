# Architecture

## Network Topology

```
Internet → Cloudflare (DNS, proxy, DDoS protection)
               └── Traefik v3.6 (reverse proxy, Let's Encrypt, HTTP/3)
                       ├── Coolify 4.0    (PaaS — deployment, env management)
                       └── n8n            (workflow automation, webhook endpoints)
                               └── PostgreSQL 16 + Redis 7
```

### Layers

**Cloudflare** — DNS resolution, DDoS mitigation, origin IP masking. All subdomains proxied (orange cloud).
- `n8n.example.com` → Hetzner server IP
- `app.example.com` → Hetzner server IP (Coolify dashboard)

**Traefik v3.6** — Reverse proxy, automatic TLS via Let's Encrypt (ACME HTTP-01), HTTP/2 + HTTP/3. Discovers services via Docker labels — no manual route config needed.
- Ports: `80` (HTTP → HTTPS redirect), `443` (HTTPS), `443/udp` (QUIC), `8080` (dashboard, internal only)
- Certificates stored in `/data/coolify/proxy/acme.json`, auto-renewed 30 days before expiry

### Network Isolation

Each service stack runs on its own Docker bridge network. Only Traefik is exposed to the internet. Databases have no published ports.

```
coolify network         → Coolify + its PostgreSQL + Redis + Soketi
n8n network (isolated)  → n8n + its PostgreSQL
```

## Services

### Coolify 4.0

Container orchestration and deployment platform. Manages Docker Compose configs, environment variables (encrypted), SSL provisioning, and git-based deployments.

**Containers:** `coolify` (PHP/Laravel), `coolify-db` (PostgreSQL 15), `coolify-redis` (Redis 7), `coolify-realtime` (Soketi WebSocket)

**Access:** `https://app.example.com:8000`

**Config location:**
```
/data/coolify/
├── source/              # Coolify application + .env
├── proxy/               # Traefik config + acme.json
│   └── dynamic/         # Dynamic routing rules
└── services/{id}/       # Per-service docker-compose + .env
```

**Commands:**
```bash
docker logs coolify -f                          # Logs
cd /data/coolify/source && docker compose restart  # Restart
docker compose pull && docker compose up -d     # Update
docker exec -it coolify-db psql -U postgres -d coolify  # DB access
```

### n8n

Workflow automation engine. Webhook endpoints, multi-API orchestration, cron scheduling, JavaScript/Python code execution.

**Containers:** `n8n-{id}` (n8n), `postgresql-{id}` (PostgreSQL 16)

**Access:** `https://n8n.example.com` (port 5678 internally)

**Key environment variables:**
```bash
N8N_EDITOR_BASE_URL=https://n8n.example.com
WEBHOOK_URL=https://n8n.example.com
GENERIC_TIMEZONE=Europe/Helsinki
DB_TYPE=postgresdb
```

Full env var reference: [configs/n8n/environment-variables.md](../configs/n8n/environment-variables.md)

**Commands:**
```bash
docker logs n8n-{id} -f                        # Logs
docker restart n8n-{id}                        # Restart
docker exec n8n-{id} n8n export:workflow --all --output=/tmp/workflows.json  # Export
docker exec postgresql-{id} pg_dump -U n8n n8n > backup.sql                 # DB backup
```

### PostgreSQL

Two isolated instances — one for Coolify (v15), one for n8n (v16). Both use Alpine images, persistent volumes, and health checks via `pg_isready`.

```bash
# Access either database
docker exec -it postgresql-{id} psql -U {user} -d {db}

# Check size
docker exec postgresql-{id} psql -U {user} -d {db} \
  -c "SELECT pg_size_pretty(pg_database_size('{db}'));"
```

## Health Checks

All containers have health checks with automatic restart:

| Service | Check | Interval | Retries |
|---------|-------|----------|---------|
| n8n | `wget -qO- http://127.0.0.1:5678/` | 5s | 10 |
| PostgreSQL | `pg_isready -U $POSTGRES_USER -d $POSTGRES_DB` | 5s | 10 |
| Traefik | `wget -qO- http://localhost:80/ping` | 4s | 5 |

```bash
# Quick status check
docker ps --format "table {{.Names}}\t{{.Status}}"

# Health details for a specific container
docker inspect {container} | jq '.[0].State.Health'
```

## Resource Usage

**Host:** Hetzner CX22 — Ubuntu 24.04 LTS, 4 GB RAM, 2 vCPU, 40 GB disk

| Stack | RAM |
|-------|-----|
| Coolify (app + db + redis + ws) | ~600 MB |
| n8n + PostgreSQL | ~400 MB |
| Traefik | ~100 MB |
| **Total** | **~1.1 GB / 4 GB** |

Storage: ~18 GB used of 40 GB. Cost: ~€7/month.

## Data Persistence

All state lives in named Docker volumes under `/var/lib/docker/volumes/`:

```
{id}_n8n-data           # n8n workflow data (/home/node/.n8n)
{id}_postgresql-data    # n8n database
coolify-db              # Coolify database
coolify-redis           # Coolify cache
```

## Backup & Recovery

**Scripts:** [`scripts/backup-procedures.sh`](../scripts/backup-procedures.sh) and [`scripts/restore-procedures.sh`](../scripts/restore-procedures.sh)

Backup covers: Coolify config (`/data/coolify/`), all PostgreSQL databases, n8n data volume, SSL certificates. 7-day retention with SHA-256 checksums.

**Recovery process:**
1. Provision new Hetzner server
2. Install Coolify
3. Restore `/data/coolify/` and Docker volumes
4. Restore databases
5. Update Cloudflare DNS to new IP
6. Restart services

**RTO:** ~30 minutes. **RPO:** depends on backup frequency (daily recommended).

## Security

- TLS everywhere — HTTP → HTTPS enforced at Traefik
- Let's Encrypt certificates, auto-renewed
- Cloudflare proxy masks origin IP
- Per-stack Docker bridge network isolation
- No database ports exposed
- UFW firewall (`22`, `80`, `443`)
- Docker socket mounted read-only where possible
- Traefik dashboard not publicly exposed

## Deployment Workflow

Adding a new service through Coolify:

1. Define service in Coolify UI (generates docker-compose.yml)
2. Coolify creates isolated Docker bridge network
3. Docker Compose brings up containers
4. Traefik discovers service via Docker labels
5. Let's Encrypt provisions TLS certificate
6. Service becomes accessible via HTTPS
