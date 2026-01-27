# Coolify Configuration

## Overview

Coolify manages Docker services through a web interface and automatically generates docker-compose configurations with proper Traefik labels for SSL and routing.

## Directory Structure

Coolify stores all configuration in `/data/coolify/`:
```
/data/coolify/
├── source/                      # Coolify application
│   ├── docker-compose.yml
│   ├── docker-compose.prod.yml
│   └── .env
├── proxy/                       # Traefik configuration
│   ├── docker-compose.yml
│   ├── acme.json               # SSL certificates
│   └── dynamic/
│       ├── coolify.yaml
│       └── default_redirect_503.yaml
└── services/                    # Deployed services
    └── {service-unique-id}/
        ├── docker-compose.yml
        └── .env
```

See full documentation in the main repository.
