# n8n Environment Variables Reference

## Core Configuration

### Base URLs
```bash
N8N_EDITOR_BASE_URL=https://n8n.example.com
WEBHOOK_URL=https://n8n.example.com
N8N_HOST=https://n8n.example.com
N8N_PROTOCOL=https
N8N_PORT=5678
```

## Database Configuration

### PostgreSQL
```bash
DB_TYPE=postgresdb
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_HOST=postgresql
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_USER=n8n_user
DB_POSTGRESDB_PASSWORD=strong_secure_password
DB_POSTGRESDB_SCHEMA=public
```

## Timezone Settings
```bash
GENERIC_TIMEZONE=Europe/Helsinki
TZ=Europe/Helsinki
```

## Execution Settings

### Execution Mode
```bash
# regular: Execute workflows in main process
# queue: Use queue mode for better scaling
EXECUTIONS_MODE=regular

# For queue mode (requires Redis)
# QUEUE_BULL_REDIS_HOST=redis
# QUEUE_BULL_REDIS_PORT=6379
```

### Execution Timeout
```bash
# Maximum execution time in seconds
EXECUTIONS_TIMEOUT=3600

# Max execution time for a single run
EXECUTIONS_TIMEOUT_MAX=7200
```

## Authentication

### Basic Auth (Simple)
```bash
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=secure_password_here
```

### Email Authentication (Production)
```bash
N8N_EMAIL_MODE=smtp
N8N_SMTP_HOST=smtp.example.com
N8N_SMTP_PORT=587
N8N_SMTP_USER=notifications@example.com
N8N_SMTP_PASS=smtp_password
N8N_SMTP_SENDER=n8n@example.com
```

## Security

### Encryption Key
```bash
# Used for encrypting credentials
# Generate with: openssl rand -base64 32
N8N_ENCRYPTION_KEY=your_32_character_encryption_key_here
```

### JWT Secret
```bash
# For API authentication
N8N_JWT_SECRET=your_jwt_secret_here
```

## Workflow Settings

### Maximum Payload Size
```bash
# In MB
N8N_PAYLOAD_SIZE_MAX=16
```

### Binary Data
```bash
# Store binary data (images, files) mode
# options: default (database), filesystem
N8N_DEFAULT_BINARY_DATA_MODE=default

# If using filesystem mode
# N8N_BINARY_DATA_STORAGE_PATH=/home/node/.n8n/binaryData
```

## External Hooks

### Webhook Configuration
```bash
# Allow HTTP webhooks (not recommended for production)
N8N_SKIP_WEBHOOK_DEREGISTRATION_SHUTDOWN=false

# Webhook testing URL
WEBHOOK_TEST_URL=https://n8n.example.com/webhook-test/
```

## Performance

### Worker Threads
```bash
# Number of worker threads
# Default: number of CPU cores
N8N_CONCURRENCY=10
```

### Memory
```bash
# Node.js memory limit
NODE_OPTIONS=--max-old-space-size=2048
```

## Logging

### Log Level
```bash
# Options: silent, error, warn, info, verbose, debug
N8N_LOG_LEVEL=info

# Log output
N8N_LOG_OUTPUT=console
```

## External Services

### API Integration
```bash
# If using n8n cloud features
# N8N_DIAGNOSTICS_ENABLED=false
# N8N_VERSION_NOTIFICATIONS_ENABLED=false
```

## Custom Settings

### User Management
```bash
# Disable user management UI (single user mode)
N8N_USER_MANAGEMENT_DISABLED=false

# Public API access
N8N_PUBLIC_API_DISABLED=false
```

### Editor Settings
```bash
# Personalization survey
N8N_PERSONALIZATION_ENABLED=false

# Templates
N8N_TEMPLATES_ENABLED=true
N8N_TEMPLATES_HOST=https://api.n8n.io
```

## Development

### Development Mode
```bash
NODE_ENV=production
N8N_RELEASE_TYPE=stable
```

## Example Complete Configuration
```bash
# Core
N8N_EDITOR_BASE_URL=https://n8n.example.com
WEBHOOK_URL=https://n8n.example.com
N8N_HOST=https://n8n.example.com
N8N_PROTOCOL=https
N8N_PORT=5678

# Database
DB_TYPE=postgresdb
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_HOST=postgresql
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_USER=n8n_user
DB_POSTGRESDB_PASSWORD=secure_password_123
DB_POSTGRESDB_SCHEMA=public

# Timezone
GENERIC_TIMEZONE=Europe/Helsinki
TZ=Europe/Helsinki

# Execution
EXECUTIONS_MODE=regular
EXECUTIONS_TIMEOUT=3600

# Security
N8N_ENCRYPTION_KEY=random_32_character_key_here_xyz
N8N_JWT_SECRET=random_jwt_secret_key

# Performance
N8N_CONCURRENCY=4
NODE_OPTIONS=--max-old-space-size=2048

# Logging
N8N_LOG_LEVEL=info
N8N_LOG_OUTPUT=console

# Features
N8N_USER_MANAGEMENT_DISABLED=false
N8N_PUBLIC_API_DISABLED=false
N8N_PERSONALIZATION_ENABLED=false

# Environment
NODE_ENV=production
N8N_RELEASE_TYPE=stable
```

## Security Best Practices

1. **Always use strong passwords** for database and authentication
2. **Generate unique encryption keys** - never use defaults
3. **Use HTTPS** - never expose n8n over plain HTTP
4. **Disable unnecessary features** in production
5. **Set appropriate execution timeouts** to prevent resource exhaustion
6. **Regularly backup** your database and encryption keys
7. **Keep n8n updated** to latest stable version

## Backup Important Variables

Always backup these variables as they're needed for data recovery:
- `N8N_ENCRYPTION_KEY` - Cannot decrypt credentials without this!
- `DB_POSTGRESDB_PASSWORD` - Database access
- `N8N_JWT_SECRET` - API authentication

Store these securely (password manager, encrypted vault).
