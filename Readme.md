docker network create frontend
```

### First-time setup
1) Create required folders:
```
mkdir local-files
mkdir postgres-data
```

2) Copy the environment file:
```
copy .env.example .env   # Windows
# or: cp .env.example .env
```
Edit `.env` and fill in values (see example below).

3) Ensure Traefik is running on the same `frontend` network:
- Example Traefik ports: `80:80` (and optionally `8080:8080` for dashboard)
- Traefik entrypoint `web` bound to `:80`
- Traefik Docker provider enabled and `exposedByDefault: false`

From your Traefik folder:
```
docker compose up -d
```

### Environment variables
Put these in `.env`:
- DOMAIN_NAME: your domain (e.g. `example.com`)
- SUBDOMAIN: subdomain for n8n (e.g. `n8n`)
- GENERIC_TIMEZONE: e.g. `Europe/London`

Postgres (used by both containers):
- POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB

n8n DB connection:
- DB_TYPE=postgresdb
- DB_POSTGRESDB_HOST=postgres
- DB_POSTGRESDB_PORT=5432
- DB_POSTGRESDB_DATABASE, DB_POSTGRESDB_USER, DB_POSTGRESDB_PASSWORD

Web settings (if not already provided via compose):
- N8N_HOST, N8N_PORT=5678, N8N_PROTOCOL=http
- WEBHOOK_URL=https://${SUBDOMAIN}.${DOMAIN_NAME}/
- NODE_ENV=production

See `.env.example` below.

### Traefik routing
In `compose.yaml` for the `n8n` service, ensure labels include:
- `traefik.enable=true`
- `traefik.http.routers.n8n.rule=Host(\`${SUBDOMAIN}.${DOMAIN_NAME}\`)`
- `traefik.http.routers.n8n.entrypoints=web`
- `traefik.http.services.n8n.loadbalancer.server.port=5678`

Note: You do not need to publish `5678` on the host when using Traefik. You can comment out the `ports` section for `n8n`.

### Run
From the project folder:
```
docker compose pull
docker compose up -d
```

### Verify
- Open: `http://SUBDOMAIN.DOMAIN_NAME`
- Optional: Traefik dashboard if enabled (default example: `http://SERVER_IP:8080`)

### Troubleshooting
- Make sure both Traefik and n8n services are attached to the same external network `frontend`.
- Confirm DNS resolves to your server.
- Check container logs:
```
docker compose logs -f n8n
docker compose logs -f postgres
```

---

## .env.example
```
# Domain config
DOMAIN_NAME=example.com
SUBDOMAIN=n8n
GENERIC_TIMEZONE=Europe/London

# Postgres (container + n8n)
POSTGRES_USER=n8n
POSTGRES_PASSWORD=change-me
POSTGRES_DB=n8n

# n8n DB connection
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=postgres
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
DB_POSTGRESDB_USER=${POSTGRES_USER}
DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}

# n8n web
N8N_HOST=${SUBDOMAIN}.${DOMAIN_NAME}
N8N_PORT=5678
N8N_PROTOCOL=http
WEBHOOK_URL=https://${SUBDOMAIN}.${DOMAIN_NAME}/
NODE_ENV=production
```
```

- Created a full `Readme.md` you can paste into the project with setup steps, required env vars, and run commands.
- Included a ready-to-use `.env.example` block.
