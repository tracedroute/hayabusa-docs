# Hayabusa disaster recovery (manual)

This stack is designed for **single-host Docker** deployment. High availability (multiple replicas, Kubernetes) is out of scope here — plan that separately when you move to K8s.

## What to back up

| Asset | Location | Tool |
|-------|----------|------|
| Application data | `/var/lib/peregrine` (Docker volume) | Host snapshots or `tar` |
| Hayabusa Auth (TOTP) | `data/auth/totp_secrets.enc` in this repo | Copy with `hayabusa/` + `.env` |
| Identity + permission history | Same volume (`manual_identity.*`, `org_permission_history.json`) | Admin → **Backup identity store** or API |
| Source + compose (no secrets) | `peregrine-src`, `docker-compose.yml`, `scripts/` | `scripts/backup-peregrine-config.sh` |
| Secrets | `.env`, `secrets/*.txt` | Copy offline; **not** in config tarball |
| Baked image | Docker Hub tag | Record `PEREGRINE_IMAGE` from `.env` |

There is **no automated backup cron** in the product — run backups on your own schedule (systemd timer, external backup agent, etc.).

## Restore order

1. Install Docker on replacement host; restore `/var/lib/peregrine` if you have a data snapshot.
2. Copy `docker-compose.yml` + `.env` (and `secrets/` if used). Run `docker compose pull && docker compose up -d`.
3. `docker compose up -d` (pull `PEREGRINE_IMAGE` from Hub).
4. If identity was corrupted: Administration → list backups → **Restore selected backup**, or `POST /api/admin/identity/restore` with a path under `/var/lib/peregrine/backups/`.
5. Run `./scripts/verify-all.sh` on the host.

## Audit / compliance

- Operator audit: hash chain in `operator_safety.json`; mirror in hayabusa-logs `operator_audit.jsonl`.
- Org admin audit: per-org JSONL under hayabusa-logs; export via org portal with optional `since` / `until`.
- Platform snapshot: `GET /api/security/compliance-export` (admin session).

## Observability after restore

- Prometheus scrapes `http://127.0.0.1:8086/metrics/hayabusa` (mirror drift, DevOps queue, rate limits).
- Alert rules: `observability/prometheus/alerts/hayabusa.rules.yml` (reload Prometheus after compose up).

## Integration apps

Custom Integration Builder apps require **code approval** before start when `HAYABUSA_INTEGRATION_REQUIRE_APPROVAL=1` (default). Re-approve after code changes.
