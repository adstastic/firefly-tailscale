# Firefly III with Tailscale

A Docker Compose setup for running [Firefly III](https://www.firefly-iii.org/) personal finance manager with [Tailscale](https://tailscale.com/) VPN integration for secure remote access.

## Features

- **Firefly III**: Full-featured personal finance manager
- **Data Importer**: Import transactions from banks and financial institutions
- **Tailscale VPN**: Secure remote access without exposing ports
- **Nginx Proxy**: Unified access through a single Tailscale endpoint
- **MariaDB**: Reliable database backend
- **Automated Cron**: Scheduled tasks for recurring transactions

## Prerequisites

- Docker and Docker Compose installed
- A Tailscale account and auth key
- Basic understanding of Docker and environment variables

## Quick Start

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/firefly-tailscale.git
   cd firefly-tailscale
   ```

2. **Copy example environment files**
   ```bash
   cp .env.example .env
   cp .db.env.example .db.env
   cp tailscale.env.example tailscale.env
   cp .importer.env.example .importer.env
   ```

3. **Configure environment variables**

   **.env** - Main Firefly III configuration:
   - Generate a 32-character `APP_KEY`: `openssl rand -base64 32 | head -c 32`
   - Generate a 32-character `STATIC_CRON_TOKEN`: `openssl rand -base64 32 | head -c 32`
   - Set `DB_PASSWORD` to a secure password

   **.db.env** - Database configuration:
   - Set `MYSQL_PASSWORD` to match `DB_PASSWORD` in `.env`

   **tailscale.env** - Tailscale configuration:
   - Get an auth key from https://login.tailscale.com/admin/settings/keys
   - Set `TS_AUTHKEY` to your Tailscale auth key
   - Optionally customize `TS_HOSTNAME`

   **.importer.env** - Data Importer configuration:
   - Set `VANITY_URL` to your Firefly III access URL
   - After starting Firefly III, create a Personal Access Token and set `FIREFLY_III_ACCESS_TOKEN`

4. **Start the services**
   ```bash
   docker compose up -d
   ```

5. **Access Firefly III**
   - Through Tailscale: `http://[tailscale-hostname]` (your configured hostname)
   - Locally: `http://localhost` (only when nginx is exposed)
   
6. **Configure Data Importer**
   - Create a Personal Access Token in Firefly III (Profile > OAuth > Personal Access Tokens)
   - Set the callback URL to `http://localhost:81/callback`
   - Uncheck "Confidential"
   - Add the token to `.importer.env`
   - Restart the importer: `docker compose restart importer`

## Architecture

```
┌─────────────────┐     ┌─────────────────┐
│    Tailscale    │     │      Nginx      │
│   (VPN/Mesh)    │────▶│  (Reverse Proxy)│
└─────────────────┘     └─────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
              ┌─────▼─────┐           ┌───────▼───────┐
              │ Firefly III│           │ Data Importer │
              │   (App)    │           │               │
              └─────┬─────┘           └───────────────┘
                    │
              ┌─────▼─────┐
              │  MariaDB  │
              │ Database  │
              └───────────┘
```

## Service URLs

When accessed through Tailscale:
- Firefly III: `http://[tailscale-hostname]/`
- Data Importer: `http://[tailscale-hostname]/importer/`

## Important Security Notes

1. **Never commit real environment files** - All `.env` files are gitignored
2. **Use strong passwords** - Generate secure passwords for database access
3. **Protect your tokens** - Keep your Tailscale auth key and Firefly access tokens secure
4. **Regular backups** - Back up your database regularly

## Troubleshooting

### Services not starting
- Check logs: `docker compose logs -f [service-name]`
- Verify all required environment variables are set
- Ensure passwords match between `.env` and `.db.env`

### Cannot access through Tailscale
- Verify Tailscale container is running: `docker compose ps tailscale`
- Check Tailscale status: `docker compose exec tailscale tailscale status`
- Ensure the device appears in your Tailscale admin console

### Data Importer not working
- Verify `FIREFLY_III_ACCESS_TOKEN` is set correctly
- Check that `FIREFLY_III_URL` is `http://app:8080` (internal Docker networking)
- Ensure the token was created with the correct callback URL

## Maintenance

### Backup database
```bash
docker compose exec db mysqldump -u firefly -p firefly > backup.sql
```

### Update containers
```bash
docker compose pull
docker compose up -d
```

### View logs
```bash
docker compose logs -f
```

## Contributing

Contributions are welcome! Please submit pull requests or open issues for any improvements.

## License

This Docker Compose configuration is provided as-is. Firefly III is licensed under the [AGPL-3.0 License](https://github.com/firefly-iii/firefly-iii/blob/main/LICENSE).