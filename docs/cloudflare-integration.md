# Cloudflare Integration

Cloudflare sits between users and the Hetzner origin server, providing DNS, DDoS protection, and TLS termination at the edge.

## Setup

### DNS Records

All services use A records pointing to the Hetzner server IP, proxied through Cloudflare:

| Type | Name | Content | Proxy |
|------|------|---------|-------|
| A | n8n | SERVER_IP | Proxied |
| A | app | SERVER_IP | Proxied |

Always use proxied mode — it hides the origin IP and enables DDoS protection.

### SSL/TLS

Set to **Full (strict)** — Cloudflare encrypts to the edge, Traefik encrypts to the origin with a valid Let's Encrypt certificate. Both ends verified.

Enable in SSL/TLS → Overview:
- Full (strict) mode
- Always Use HTTPS
- Automatic HTTPS Rewrites
- TLS 1.3

### Network Settings

Enable under Network:
- HTTP/2 (default)
- HTTP/3 (QUIC)
- WebSockets (required for Coolify real-time)

## Webhook Cache Bypass

n8n webhook endpoints should not be cached. Add a page rule:

```
URL: n8n.example.com/webhook/*
Cache Level: Bypass
```

## Security

- **Bot Fight Mode** — enable under Security → Bots (free)
- **DNSSEC** — enable under DNS → Settings
- **Security Level** — Medium (default, good baseline)

## Troubleshooting

### Redirect Loop (ERR_TOO_MANY_REDIRECTS)

SSL/TLS mode is set to Flexible while Traefik also forces HTTPS → infinite loop. Fix: set mode to Full or Full (strict).

### 502 Bad Gateway

Origin is down. Check:
```bash
ssh root@SERVER_IP
docker ps                          # Are containers running?
docker logs coolify-proxy          # Is Traefik healthy?
```

### 525 SSL Handshake Failed

Traefik hasn't issued a certificate yet, but Cloudflare expects one (Full strict mode).
```bash
docker logs coolify-proxy | grep acme   # Check Let's Encrypt logs
```
Temporarily switch to Full mode, wait for cert, then switch back to Full (strict).

### DNS Not Resolving

```bash
dig n8n.example.com +short
# Should return Cloudflare IPs (104.x.x.x or 172.x.x.x), not your server IP
```

If it returns your server IP, the record isn't proxied. If it returns nothing, DNS hasn't propagated yet (can take up to 48h after nameserver change).
