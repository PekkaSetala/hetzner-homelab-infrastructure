# hetzner-homelab-infrastructure

Self-hosted AI workflow automation platform on Hetzner Cloud with Cloudflare at the edge.

## Architecture

```
Internet → Cloudflare (DNS + proxy)
               └── Traefik (reverse proxy, SSL termination)
                       ├── Coolify   (PaaS management)
                       ├── n8n       (workflow automation)
                       └── Open WebUI (LLM interface)
                               └── PostgreSQL + Redis
```

All services run in isolated Docker bridge networks. Databases have no exposed ports.

## Stack

| Component | Version | Role |
|-----------|---------|------|
| Traefik | v3.6 | Reverse proxy, automatic SSL via Let's Encrypt |
| Coolify | 4.0 | Self-hosted PaaS, container management |
| n8n | latest | Workflow automation engine |
| Open WebUI | latest | LLM interface |
| PostgreSQL | 16 | Persistent storage |
| Redis | 7 | Caching |
| Cloudflare | — | DNS, DDoS protection, IP masking |

**Host:** Hetzner VPS — Ubuntu 24.04 LTS, 4 GB RAM (~1.7 GB utilized, 8 services)  
**Cost:** ~€6.83/month

## Workflows

**AI Motorcycle Visual Identifier** — Webhook receives an image → OpenAI Vision analyzes it → recommendation engine → email sent. ([docs](docs/workflows/motorcycle-identifier.md))

**Selkokielelle — Plain Language Converter** — Web form submission triggers a GPT-4 pipeline that rewrites Finnish text into plain language for accessibility. ([docs](docs/workflows/selkokielelle-converter.md))

## Security

- HTTP → HTTPS enforced at Traefik
- Let's Encrypt certificates, auto-renewed
- Cloudflare masks origin IP
- Docker bridge network isolation per service group
- UFW firewall

## Docs

- [Getting Started](GETTING-STARTED.md)
- [Architecture](docs/architecture.md)
- [Deployment Guide](docs/deployment-guide.md)
- [Services](docs/services.md)
- [Cloudflare Integration](docs/cloudflare-integration.md)

## License

MIT
