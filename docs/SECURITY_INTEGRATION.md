# Wazuh & Shuffle integration (Option A)

Hayabusa exposes Wazuh SIEM and Shuffle SOAR behind the same session as the rest of the product. Users sign in once; `/wazuh/` and `/shuffle/` are authenticated reverse proxies. Upstream UIs stay on **127.0.0.1** on each Hayabusa host and are not meant to be opened directly.

This document is operational guidance, not legal advice.

## Deployment model: local Hayabusa + public domain

Each customer runs a **locally hosted** Hayabusa instance (your image or installer on their hardware/VPC). End users and integrators reach that instance at a **public HTTPS URL** (customer-owned domain or a subdomain you assign, e.g. `https://acme.example.com`).

```
                    Internet / customer LAN
                              │
                    TLS (443) │  Traefik / nginx
                              ▼
              https://customer-domain/  ──►  Hayabusa (Peregrine)
                    │              │
         /wazuh/    │              │  /shuffle/
         (proxy)    │              │  (proxy)
                    ▼              ▼
              127.0.0.1:5601   127.0.0.1:3001 / :5001
              Wazuh dashboard  Shuffle UI / API
```

**What users see:** one login at `https://customer-domain/`, then Watch → Wazuh/Shuffle at `https://customer-domain/wazuh/` and `/shuffle/`.

**What stays local:** Wazuh manager, indexer, Shuffle, and API ports bind to loopback on that host. Only Hayabusa’s HTTPS entrypoint should be exposed on the WAN.

### Required settings per instance

| Variable | Purpose |
|----------|---------|
| `PRODUCTION_DOMAIN` | Hostname on TLS cert / routing |
| `OAUTH_PUBLIC_BASE_URL` | Canonical `https://` base (redirects, cookies, proxy rewrites) |
| `PEREGRINE_TRUST_PROXY=1` | Honor `X-Forwarded-*` from edge proxy (default on) |
| `PEREGRINE_WAZUH_INTERNAL_URL` | `https://127.0.0.1:5601` (not the public domain) |
| `PEREGRINE_SHUFFLE_UI_INTERNAL_URL` | `http://127.0.0.1:3001` |
| `PEREGRINE_SHUFFLE_API_INTERNAL_URL` | `http://127.0.0.1:5001` |

Do **not** point `PEREGRINE_*_INTERNAL_*` at the public domain; those are server-side upstreams on the same machine as Hayabusa.

### Integrators and automation

`GET /api/security/status` (Hayabusa session) returns relative `links` plus `absolute_links` when a public base is known (`OAUTH_PUBLIC_BASE_URL` or the request host behind the proxy). Use `integration.wazuh_agent_group` / `integration.shuffle_org_slug` to scope agents and workflows per tenant.

**Wazuh agents** enroll to the **manager** on the customer host (typically TCP 1514/1515), not through `/wazuh/`. Publish manager agent ports in `docker-compose.security.yml` only if agents connect from outside the host; keep the dashboard on 127.0.0.1.

**Shuffle workers / webhooks** call the customer’s Hayabusa URL or localhost API paths as you document for that product; the Shuffle UI for humans remains `/shuffle/` behind Hayabusa auth.

## Architecture

| Surface | Hayabusa path | Upstream (localhost) | Auth |
|--------|---------------|----------------------|------|
| Wazuh dashboard | `/wazuh/` | `PEREGRINE_WAZUH_INTERNAL_URL` (e.g. `https://127.0.0.1:5601`) | OpenSearch **proxy** auth: `x-hayabusa-user`, `x-hayabusa-roles` |
| Shuffle UI | `/shuffle/` | `PEREGRINE_SHUFFLE_UI_INTERNAL_URL` | Hayabusa session only |
| Shuffle API | `/shuffle/api/v1/` | `PEREGRINE_SHUFFLE_API_INTERNAL_URL` | Server-side `Authorization` + `Org-Id` (key not in browser) |
| Wazuh API (alerts) | `/api/security/infrastructure-alerts` | `PEREGRINE_WAZUH_API_URL` | Service credentials in `.env` |

Pattern matches existing **Grafana** integration: unmodified upstream images, separate containers, Hayabusa proprietary code only proxies and calls APIs.

## Commercial use of Hayabusa with Wazuh / Shuffle / Grafana / MongoDB

Hayabusa (proprietary) does **not** embed or link Wazuh, Shuffle, or Grafana
source into the Peregrine binary. It:

- Runs **official upstream container images** side by side (compose).
- Proxies HTTP to those services after Hayabusa login (`/wazuh/`, `/shuffle/`, `/grafana/`).
- Mounts **configuration only** where needed (for example Grafana provisioning
  datasources/dashboards JSON under `observability/grafana/provisioning/`).
  That is **not** a modification of Grafana/Shuffle program source.

### Are we modifying AGPL software?

**No.** As of 2026-09-30 this deployment:

- Pulls unmodified `grafana/grafana:11.2.2` (and Shuffle GHCR images when the
  security profile is up).
- Does **not** carry Grafana or Shuffle source trees or patches in this repo.
- Only binds config/provisioning files and runtime data volumes.

AGPL’s network source-offer clause is aimed at **modified** versions of the
AGPL program. Running stock upstream images with config mounts is not the same
as shipping a modified AGPL derivative. You still owe attribution and should
keep notices current; get counsel if you change that model (forks, patches,
or embedding their UI into Hayabusa’s binary).

### Are we “hosting” them legally / right now?

| Component | Running on this production host (`hayabusa.tracedroute.net`) | Posture |
|-----------|---------------------------------------------------------------|---------|
| **Grafana** | **Yes** (`hayabusa-grafana`, proxied at `/grafana/`) | You operate unmodified Grafana as a networked service for Hayabusa users. Prefer AGPL notices + link to upstream source (done in `THIRD_PARTY_NOTICES.md`). |
| **Shuffle** | **Configured** via `COMPOSE_PROFILES=security`, but containers were **not running** at last check | When enabled, same proxy model. AGPL applies to Shuffle itself; Hayabusa remains separate. |
| **MongoDB 4.4** (Shuffle dep) | Only when Shuffle stack is up | **SSPL** — not AGPL. SSPL restricts offering MongoDB *as a service*. Keeping stock `mongo:4.4` solely as Shuffle’s private DB is the intended model; counsel if you productize Mongo separately. |
| **Wazuh** | Same security profile (not running at last check) | GPLv2 stack in separate containers; proxy only. |

**Customer self-host vs TracedRoute-operated Core:** If customers run the full
compose on their own hosts, *they* operate those containers. If *you* run Core
at `hayabusa.tracedroute.net` for end users (current Grafana case), *you* are
the operator of that Grafana instance and should keep AGPL/GPL/SSPL notices and
upstream source pointers accurate.

| Product | Typical license | Who runs it in this model | Practical note |
|---------|-----------------|---------------------------|----------------|
| **Hayabusa** | Your commercial license | You license; instance may be customer- or TracedRoute-operated | Proxy/integration code stays proprietary; no Wazuh/Shuffle/Grafana source in the Peregrine image. |
| **Grafana** | AGPL-3.0 | Operator of the compose host | Unmodified upstream image + provisioning JSON only. |
| **Wazuh** | GPLv2 (stack) | Operator of the compose host | Official images; GPLv2 applies to Wazuh redistribution, not to Hayabusa as a separate product. |
| **Shuffle** | AGPL-3.0 | Operator of the compose host | Unmodified upstream images; AGPL notices/source for Shuffle itself. |
| **MongoDB** | SSPL-1.0 | Shuffle dependency only | Do not rebrand/resell Mongo as your database product without counsel. |

**Not in Hayabusa:** modifying Wazuh/Shuffle/Grafana source, static linking, or shipping their UI inside the Peregrine image as a derivative work.

**You should:** document third-party licenses (`peregrine-src/THIRD_PARTY_NOTICES.md`), ship compose/env templates so deployments use **unmodified** upstream images, publish a GPL source-offer path (`scripts/gpl-source-offer.sh`), and revisit counsel if you **fork** AGPL components or expand managed hosting.

## Setup

1. Certs and `docker compose -f docker-compose.yml -f docker-compose.security.yml --profile security up -d`
2. `./scripts/apply-wazuh-hayabusa-proxy-security.sh` (after indexer is healthy)
3. Set `.env` internal URLs and API secrets (see `.env.example`)
4. `./scripts/deploy-security-ops.sh` (after Peregrine container changes)
5. `./scripts/fix-wazuh-vulnerability-indexer.sh` (idempotent; run automatically from deploy script if manager is up)
6. `./scripts/verify-security-integration.sh` (smoke-check upstreams + Peregrine markers)

### Production hardening (optional)

| Variable | Purpose |
|----------|---------|
| `PEREGRINE_WAZUH_CA_CERT` | Path to Wazuh root CA inside Peregrine (default `/etc/peregrine/wazuh-ca.pem` when compose volume is mounted) |
| `PEREGRINE_WAZUH_TLS_VERIFY=1` | Require TLS verification for Wazuh HTTPS upstreams |
| `PEREGRINE_PRODUCTION=1` | Log warnings for weak security-ops config at startup |
| `PEREGRINE_REQUIRE_PROD_SECRETS=1` | **Fail startup** if Wazuh/Shuffle secrets or TLS verify are missing |
| `SESSION_COOKIE_SECURE=auto` | **Default.** Session cookie gets `Secure` only when the browser uses HTTPS; plain HTTP LAN (`:8086`) still works |
| `PEREGRINE_LOCALHOST_USE_PUBLIC_BASE=1` | Optional: rewrite localhost proxy assets to `OAUTH_PUBLIC_BASE_URL` (tunnel/dev only) |

Hayabusa serves **HTTP and HTTPS concurrently** when both entrypoints exist (e.g. `http://192.168.1.221:8086` and `https://your-domain/`). Keep `PEREGRINE_TRUST_PROXY=1` behind nginx/Traefik so HTTPS scheme is detected via `X-Forwarded-Proto`. Wazuh/Shuffle proxy cookies follow the same rule automatically.

Hayabusa admins can export a compliance snapshot (config summary, service health, alert counts, audit chain status) via `GET /api/security/compliance-export`.

Operator audit entries appended after this update include a SHA-256 hash chain (`prev_hash` / `entry_hash` on each row in `operator_safety.json`).

## Tenant scoping

- **Wazuh:** agent group `hayabusa_{tenant_key}`; proxy roles: Hayabusa system admin → `admin`, else `kibanauser` only
- **Shuffle:** Hayabusa system admin → `PEREGRINE_SHUFFLE_API_KEY`; other users → `PEREGRINE_SHUFFLE_USER_API_KEY` (role `user`)

## Verify

1. Log into Hayabusa → **Watch** → Wazuh / Shuffle landing pages → **Open** (same window).
2. `/wazuh/` loads dashboard without a Wazuh login form.
3. `/shuffle/` loads SOAR without Shuffle login; workflows call `/shuffle/api/v1/…`.

If Wazuh still prompts for login, re-run `apply-wazuh-hayabusa-proxy-security.sh` and confirm `opensearch_dashboards.yml` has `auth.type: proxy`.
