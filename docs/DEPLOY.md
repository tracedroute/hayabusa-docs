# Hayabusa deploy (two files)

Copy to any Linux host with Docker:

1. `docker-compose.yml`
2. `.env` (from `.env.example`, fill secrets)

```bash
chmod 600 .env
docker compose pull
docker compose up -d
```

Images (set in `.env`):

- `PEREGRINE_IMAGE` — main app (Docker Hub)
- `HAYABUSA_LOGS_IMAGE` — centralized audit/log sidecar (Docker Hub)

Optional Wazuh + Shuffle (needs extra certs under `security/wazuh/` on that host):

```bash
docker compose -f docker-compose.yml -f docker-compose.security.yml --profile security up -d
```

Development of the app still uses the full `website-peregrine` git tree; production hosts only need the two files above.
