# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Docker-based deployment solution for Firefly III (personal finance manager) integrated with Tailscale VPN for secure remote access. The project uses Docker Compose to orchestrate multiple services including Firefly III, MariaDB, Nginx reverse proxy, and Tailscale VPN.

## Key Commands

### Initial Setup
```bash
# Copy and configure environment files
cp .env.example .env
cp .db.env.example .db.env
cp tailscale.env.example tailscale.env
cp .importer.env.example .importer.env

# Generate required 32-character tokens
openssl rand -base64 32 | head -c 32  # For APP_KEY and STATIC_CRON_TOKEN
```

### Docker Operations
```bash
# Start all services
docker compose up -d

# View logs for specific service
docker compose logs -f [service-name]  # Services: tailscale, nginx, app, db, importer, cron

# Stop all services
docker compose down

# Update containers to latest versions
docker compose pull
docker compose up -d

# Check service status
docker compose ps

# Execute commands in containers
docker compose exec app php artisan [command]
docker compose exec db mariadb -u root -p
```

## Architecture

The project consists of 6 Docker services:

1. **tailscale**: VPN mesh network for secure remote access
2. **nginx**: Reverse proxy (runs in Tailscale's network namespace)
3. **app**: Firefly III web application
4. **db**: MariaDB database
5. **importer**: Data import tool for bank transactions
6. **cron**: Scheduled task runner for recurring transactions

### Network Architecture
- Nginx uses Tailscale's network namespace (`network_mode: service:tailscale`)
- All internal services communicate via `firefly_iii` Docker network
- No ports are exposed to the host except through Tailscale VPN
- Access URLs:
  - Main app: `http://firefly-mac/` (via Tailscale)
  - Importer: `http://firefly-mac/importer/` (via Tailscale)

### Critical Configuration Rules
1. **Database passwords** in `.env` (DB_PASSWORD) and `.db.env` (MYSQL_PASSWORD) must match exactly
2. **APP_KEY** must be exactly 32 characters
3. **STATIC_CRON_TOKEN** must be exactly 32 characters
4. **Tailscale hostname** in `tailscale.env` (TS_HOSTNAME) must match the URL in `.env` (APP_URL)

## Environment Files

All environment files are gitignored. When modifying:
- `.env`: Main Firefly III configuration
- `.db.env`: MariaDB configuration
- `tailscale.env`: Tailscale VPN settings
- `.importer.env`: Data importer settings

## Testing Changes

```bash
# Validate Docker Compose configuration
docker compose config

# Test Nginx configuration changes
docker compose exec nginx nginx -t
docker compose restart nginx

# Check service connectivity
docker compose exec nginx curl -I http://app:8080
docker compose exec nginx curl -I http://importer:8080
```

## Common Development Tasks

### Updating Firefly III
```bash
docker compose pull app
docker compose up -d app
docker compose exec app php artisan cache:clear
```

### Database Operations
```bash
# Backup database
docker compose exec db mariadb-dump -u root -p firefly > backup.sql

# Access database
docker compose exec db mariadb -u firefly -p firefly
```

### Debugging Issues
```bash
# Check all service logs
docker compose logs

# Inspect network configuration
docker network inspect firefly-tailscale_firefly_iii

# Check Tailscale status
docker compose exec tailscale tailscale status
```

## File Structure

```
.
├── docker-compose.yml          # Main deployment configuration
├── docker-compose-official.yml # Reference config without Tailscale
├── nginx.conf                  # Reverse proxy configuration
├── .env.example               # Firefly III config template
├── .db.env.example            # Database config template
├── tailscale.env.example      # Tailscale config template
├── .importer.env.example      # Importer config template
└── README.md                  # Comprehensive setup documentation
```

## Important Notes

- This is a deployment/infrastructure project with no application code
- All services run as containers - no local development environment needed
- Persistent data stored in Docker volumes (firefly_iii_upload, firefly_iii_db)
- The project prioritizes security through VPN-only access
- When troubleshooting, always check that environment variables match between related services