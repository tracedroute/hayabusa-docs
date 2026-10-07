# Hayabusa Zero-Touch Provisioning (ZTP) & Controller Architecture

**Audience:** operators, integrators, and developers working on Hayabusa Core + Controller.  
**Scope:** how site ZTP, Fleet Incoming, GitOps recipe sources, and the Core↔Controller bridge fit together.  
**Lab pins (as of 2026-09-30):** Core live image may vary; Controller bridge identity in lab often `admin@100.64.0.7` / `hayabusa-controller2#2`. Prefer documenting the connected `user_key` from `CONTROLLER_REGISTRY.list_connected()` rather than hard-coding tags.

This document describes **current intended product behavior** as implemented in:

- Core: `peregrine-src/` (especially `run.py`, `hayabusa_gitops_agent.py`)
- Controller: `controller-hotfixes/` (especially `ztp_edge.py`, `bridge_rpc.py`, `app/main.py`)
- Ops: `docker-compose.yml`, `docker-compose.controller.yml`, `.env` image pins
- Related: `controller-hotfixes/docs/GITOPS.md`, `docs/IMAGE-NEST-AND-WINDOWS-IMAGES.md`

---

## 1. One-sentence model

**Hayabusa Core is the product brain** (Fleet, claim, GitOps pull, orchestration).  
**Hayabusa Controller is the site LAN edge** (DHCP/discovery, `/ztp/fetch` delivery, secrets, job approve/hydrate).  
**Git (GitLab/GitHub) is the source of truth for Advanced mode scripts** — it is *not* the runtime pipe that boots a host on the LAN.

```
                  GitLab / GitHub (Advanced)
                         │ webhook / pull
                         ▼
┌──────────────────────────────────────────────────────────┐
│  Hayabusa Core (hub)                                     │
│  UI :8086 · Fleet Incoming · GitOps agent · WS :8791     │
│  Hub DHCP: PERMANENTLY OFF                               │
└────────────▲───────────────────────────────┬─────────────┘
             │ bridge RPC / presence         │ push workspace
             │ (ztp.leases, ztp.recipes, …)  │ iac.import_sync
┌────────────┴───────────────────────────────▼─────────────┐
│  Hayabusa Controller (site appliance)                    │
│  API :8790 · dnsmasq DHCP/PXE · /ztp/fetch · secrets     │
│  Workspace: Ansible/ztp + Ansible/bare-metal-ztp         │
└────────────▲─────────────────────────────────────────────┘
             │ DHCP lease + HTTP fetch
             ▼
        Blank / PXE / vendor ZTP host on site LAN
```

---

## 2. Roles and ownership

### 2.1 Hayabusa Core (hub)

| Owns | Does not own |
|------|----------------|
| Fleet Incoming UI and claim APIs | Site L2 DHCP for customer LANs |
| Mapping controller leases → Incoming rows | Serving day-to-day ZTP payloads on the wire (when a controller branch is selected) |
| GitOps clone/pull and merge of ZTP repos | Long-lived Git PATs (those live in the controller vault) |
| Pushing synced workspace onto the controller | Binding UDP 67 on the hub |
| Mesh/control WebSocket server (`:8791`) | Customer LAN interface selection |
| Refusing hub DHCP enable | |

Primary code: `_ztp_refuse_core_hub_dhcp`, `_ztp_pending_from_controller_leases`, `_ztp_controller_rpc`, `api_my_controller_ztp*`, Fleet claim/intent routes in `peregrine-src/run.py`; `hayabusa_gitops_agent.py`.

### 2.2 Hayabusa Controller (site)

| Owns | Does not own |
|------|----------------|
| dnsmasq DHCP (and optional TFTP/PXE) on a LAN NIC | Hub Fleet claim policy |
| Auth-exempt `GET /ztp/fetch/{path}` for booting devices | Being the GitOps webhook target (webhooks hit Core) |
| Seeking journal (`seeking.json`) | Deciding org/user claim rights |
| Discovering recipes from local workspace trees | |
| Secrets vault, job queue approve/hydrate | |
| Bridge client to Core WS | |
| Novice vs Advanced setup + GitOps config UI | |

Primary code: `ZtpEdge` in `controller-hotfixes/ztp_edge.py` (and `app/ztp_edge.py`); HTTP routes and lifespan auto-start in `app/main.py`; RPC in `bridge_rpc.py`.

### 2.3 Mental model for operators

1. A device PXE/DHCP-boots on the **site** network.  
2. The **controller** hands it an address (and optionally boot filename / next-server).  
3. The device HTTP-fetches a script/config from the **controller** (`/ztp/fetch/...`).  
4. The **controller** journals that fetch (“seeking”).  
5. Core periodically/on UI load pulls leases (+ seeking) over the **bridge** and shows **Fleet → Incoming**.  
6. An operator **claims** the host and attaches a recipe **before** full config delivery completes.  
7. In Advanced mode, recipe *content* originated from Git; Core pulled it and pushed it into the controller workspace.

---

## 3. Network and ports

| Port | Service | Typical lab bind |
|------|---------|------------------|
| **8086** | Core UI / API (Gunicorn/Flask) | `http://192.168.1.221:8086` |
| **8790** | Controller HTTP API + dashboard + `/ztp/fetch` | `http://192.168.1.221:8790` |
| **8791** | Core WebSocket bridge for controllers | `ws://192.168.1.221:8791` |
| **UDP 67** | Site DHCP (controller dnsmasq, when enabled) | Bound to LAN NIC (e.g. `ens18`) |
| **UDP 69** | Optional TFTP (controller), often off if another PXE stack holds it | Optional |

### 3.1 Critical deployment rules

1. **Core and Controller may share a host** (both often use host networking). Only **one** process may own UDP 67 — that must be the **controller** ZTP edge, never Core hub DHCP.
2. **Bridge WebSocket must reach Core on the LAN** in lab/split setups:  
   `HAYABUSA_WS_URL=ws://192.168.1.221:8791`  
   First-boot defaults that point at `127.0.0.1` or a CGNAT mesh-only URL frequently fail; prefer LAN.
3. Controller public base URL for fetch links is typically:  
   `http://<controller-lan-ip>:8790` with paths under `/ztp/fetch/...`.

### 3.2 UI path rewrite (controller branch selected)

When an operator selects a controller branch in Core Fleet, the browser rewrites hub ZTP API paths to **my-controller** proxies (see `peregrine-controller-branch.js` → `rewriteZtpPathForController`), e.g.:

- `/api/ztp/status` → `/api/my-controller/ztp/status`
- `/api/ztp/config` / `/api/ztp/zerotouch-config` → `/api/my-controller/ztp/config`
- `/api/ztp/interfaces` / `/api/ztp/host-interfaces` → `/api/my-controller/ztp/interfaces`
- `/api/ztp/leases` / `/api/dhcp/leases` → `/api/my-controller/ztp/leases`
- `/api/dhcp/start|stop` / `/api/ztp/start|stop` → `/api/my-controller/ztp/start|stop`
- `/api/ztp/pending-devices` → `/api/my-controller/ztp/pending-devices`
- Related DHCP-request tails similarly

Policy treats `/api/my-controller/ztp/*` as controller-delegated; classic `/api/ztp/*` and `/api/dhcp/*` stay gated when a controller is bound so the UI must use the rewrite. Hub DHCP enable remains refused (`core_hub_dhcp_disabled`).

---

## 3.3 Robotics / SDR / MAVLink (controller edge)

**Seen/heard on Core:** browser URLs stay `/api/robotics/*` (and disaster SDR stream URLs).  
**Transmitted from controller:** when a branch is selected and live, Core calls controller RPCs:

| RPC | Role |
|-----|------|
| `edge.capabilities` | SDR / dump1090 / mavlink libs present |
| `edge.mavlink.*` | connect / status / command / mission / joystick |
| `edge.sdr.*` | start / stop / status / control / audio_chunk |
| `edge.adsb.tracks` | local dump1090 JSON (or `unavailable`) |

Fail closed when the bound controller is offline (`code: controller_required`). Without a bound controller, Core may use a local break-glass path for lab installs.

---

## 3.4 Integration Builder sync

Webhooks, custom apps (`/api/custom-apps`), and automation jobs persist on Core and are mirrored to the selected controller via:

- UI: Integrations → **Sync with controller**
- API: `POST /api/my-controller/integrations/sync` → `integrations.bundle.get` / `integrations.bundle.put`

Builder code approval and start still follow Core policy (`HAYABUSA_INTEGRATION_REQUIRE_APPROVAL`); the controller holds the synced bundle for the site.

---

## 4. Controller ZTP edge (deep dive)

### 4.1 Data directory layout

Under `$CONTROLLER_DATA_DIR` (default `/var/lib/hayabusa-controller`):

```
ztp-edge/
  config.json          # enabled, interface, subnet, range, gateway, dns, PXE flags, fetch base
  runtime/             # dnsmasq.conf, pid, leases, logs
  tftp/                # optional boot files
  fetch/               # optional drop-in fetch root
  seeking.json         # fetch journal (bounded)
iac/workspace/
  Ansible/ztp/              # vendor ZTP scripts / dispatch trees
  Ansible/bare-metal-ztp/   # bare-metal recipe packs
  Ansible/...               # other playbooks
  OpenTofu/...              # IaC
bootstrap.env          # first-boot admin password, bridge token, WS URLs (secret)
state/enrollment.json  # enrollment / WS hints
state/gitops.json      # GitOps + optional ztp_repo fields
secrets/               # vault (PATs, webhook secrets, SSH, …)
```

Packaged defaults ship under `controller-hotfixes/devops_iac_defaults/` and are rsynced into the workspace on first boot **when GitOps is not enabled**.

### 4.2 Configuration (`ZtpEdge` / `DEFAULT_CONFIG`)

Typical fields (names illustrative; see `ztp_edge.py`):

| Field | Meaning |
|-------|---------|
| `enabled` | Persist intent to run DHCP after start; used for **auto-start on boot** |
| `interface` | LAN NIC (e.g. `ens18`) |
| `subnet` / `netmask` / `range_start` / `range_end` | DHCP pool |
| `gateway` / `dns` | Options offered to clients |
| `lease_hours` | Lease lifetime |
| `next_server` / `boot_filename` | PXE next-server + bootfile when PXE enabled |
| `enable_tftp` / `enable_pxe` | Optional; often **false** when another PXE engine owns UDP 69 |
| `hayabusa_ztp_fetch_base` | Base URL advertised/used for fetches (e.g. `http://192.168.1.221:8790/ztp/fetch`) |

HTTP:

- `GET /api/ztp/status`, `GET|PUT /api/ztp/config`
- `POST /api/ztp/start`, `POST /api/ztp/stop`
- `GET /api/ztp/leases`, `GET /api/ztp/interfaces`, `GET /api/ztp/recipes`

Starting/stopping DHCP requires an authenticated session with RBAC permission **`approve_ztp`**.

### 4.3 dnsmasq behavior

- Rendered conf: `port=0` (DHCP-only DNS off), `bind-interfaces`, lease file, optional TFTP root and `dhcp-boot`.
- Process kept in foreground under supervisor logic; PID recorded for stop/restart.
- **Vendor class (option 60) and user class (option 77) are not required in conf** for discovery. The edge **parses dnsmasq logs** for DISCOVER/REQUEST lines and attaches `vendor_class` / `user_class` hints onto leases and `dhcp_events`.

### 4.4 Leases and DHCP events

- `leases()` reads dnsmasq’s lease file and merges log-derived class hints.
- `dhcp_events()` exposes recent DISCOVER/REQUEST-style events for Fleet/ops visibility.
- Example lab lease: MAC `bc:24:11:9a:83:01` → IP on the site ZTP subnet (e.g. `10.240.0.158`).

### 4.5 `/ztp/fetch` delivery

- Route: **`GET /ztp/fetch/{path}`** — **auth-exempt** and **CSRF-exempt** by design (booting devices have no browser session).
- Path resolution order (high level): dedicated `fetch_root`, then workspace trees `Ansible/ztp`, `Ansible/bare-metal-ztp`, related OpenTofu/ztp / legacy roots. Path traversal is blocked.
- Every hit calls **`record_fetch`**, which updates **`seeking.json`**.

Examples:

```http
GET http://192.168.1.221:8790/ztp/fetch/generic/dispatch.sh
GET http://192.168.1.221:8790/ztp/fetch/arista/710p-7200s-dispatch
```

Successful file serve → seeking status like `seeking_config` / awaiting full config.  
Missing file → rejected / config rejected outcomes.

**Important:** A successful fetch alone does **not** mark configuration as fully **delivered** for Fleet claim purposes. Delivery completion is a later provisioning/claim-path concern; seeking leaves room for “still seeking full config.”

### 4.6 Seeking journal

File: `ztp-edge/seeking.json` (size-bounded, e.g. ~200 records).

Typical fields per record: `ip`, `mac`, `path` / `dispatch_path`, `hits`, `status`, `config_status`, `config_outcome`, timestamps.

Exposed over bridge as **`ztp.seeking`**.

### 4.7 Recipe discovery (`list_recipes`)

Returns two families:

1. **`vendor_ztp`** — files/trees under vendor ZTP paths (`Ansible/ztp`, …), each with `id`, `name`, `vendor`, `path`, `kind: vendor_ztp`.
2. **`bare_metal`** — recipe packs under `Ansible/bare-metal-ztp` (dirs/JSON), `kind: bare_metal`.

HTTP: `GET /api/ztp/recipes`  
RPC: `ztp.recipes` → includes metadata:

- `source: "controller"` — Novice / local workspace is authority  
- `source: "gitops"` — Advanced; GitOps/`ztp_repo` active (content still *served* from the materialized controller workspace)

### 4.8 Auto-start on container recreate

If `config.enabled` is true and dnsmasq is not running, controller **lifespan** calls `ztp_edge.start()` on boot (added so Hub pulls + recreate do not silently leave DHCP down). Present from controller image **`09152026.2`**.

---

## 5. Mode-based recipe sources (Novice vs Advanced)

### 5.1 Novice

- Chosen in controller first-run / setup redo.
- GitOps **disabled**.
- Packaged defaults (and local edits on the controller) are the source of truth under `iac/workspace`.
- Recipe listing → `source: controller`.
- Operators edit Ansible/ZTP trees on the controller (or sync from Core DevOps tooling); no dedicated ZTP Git repo required.

### 5.2 Advanced

- Setup requires a **ZTP scripts repository** in addition to the IaC repo:
  - `ztp_repo` (required)
  - `ztp_ref` (branch/tag)
  - `ztp_path_prefix` (optional subdirectory)
- Plus normal GitOps fields for the IaC repo (`repo`, `ref`, `path_prefix`, provider, vault keys, webhook secret key).
- Local controller workspace becomes **read-only for UI edits** while GitOps is enabled — change Git, not the live tree.
- Core GitOps agent:
  1. Pulls IaC repo (shallow).
  2. If `ztp_repo` differs from the IaC repo, pulls the ZTP repo and **merges** into workspace `Ansible/ztp` and `Ansible/bare-metal-ztp`.
  3. Pushes the combined workspace to the controller via **`iac.import_sync`** (packaged console defaults are never overwritten).
- Recipe listing then reports `source: gitops` when that path is active.

See also: `controller-hotfixes/docs/GITOPS.md`.

### 5.3 What “source of truth” does *not* mean

Git is **not** contacted by the PXE client.  
At boot time the host only speaks DHCP/HTTP to the **controller**. Git must already have been pulled and pushed onto that workspace.

---

## 6. Bridge: Core ↔ Controller

### 6.1 Connection

- Controller opens a WebSocket to Core (`HAYABUSA_WS_URL`, often also mesh/fallback variants).
- After hello/grant, Core registry shows a connected session, e.g. `admin@100.64.0.5`, with capabilities including **`ztp-edge`**.
- Lab note: logging into the controller as **manual `admin`** starts the bridge bound to that identity; Fleet “my-controller” paths must match the signed-in Core user / selected controller.

### 6.2 ZTP-related RPC methods

| Method | Purpose |
|--------|---------|
| `ztp.status` | Running?, pid, config summary, lease count |
| `ztp.config.get` / `ztp.config.put` | Read/write edge config |
| `ztp.start` / `ztp.stop` | Start/stop dnsmasq (may enqueue jobs) |
| `ztp.leases` | Current leases (+ class hints) |
| `ztp.dhcp_events` | Recent DISCOVER/REQUEST-style events |
| `ztp.seeking` | Fetch journal records |
| `ztp.recipes` | Vendor + bare-metal recipe lists + `source` |
| `ztp.interfaces` | Candidate LAN interfaces |

Core invokes these through `_ztp_controller_rpc` / controller registry `rpc`, requiring an authenticated Core session whose identity matches a live controller connection (`_my_controller_require_user`).

### 6.3 Related non-ZTP RPCs (context)

GitOps and IaC also use the bridge (`gitops.credentials`, `iac.import_sync`, job hydrate/approve, secrets). ZTP Advanced depends on that same plane for pulling and pushing recipe trees.

---

## 7. Core Fleet Incoming (pending relay)

### 7.1 Building Incoming from leases

Function: **`_ztp_pending_from_controller_leases`** (`run.py`).

Flow:

1. RPC `ztp.leases` (required).
2. Enrich with `ztp.seeking` (and Core-side seeking helpers where applicable).
3. Emit Incoming-shaped devices, including:

| Field | Typical value / meaning |
|-------|-------------------------|
| `discovery_source` | `dhcp_lease` |
| `discovery_via` | `controller_edge` |
| `mac` / `ip` / `hostname` | From lease |
| `dhcp_vendor_class` / `dhcp_user_class` | From log hints (e.g. `PXEClient…`, `iPXE`) |
| `status` / `config_status` | e.g. `seeking_config`, awaiting full config |
| `controller_key` | e.g. `admin@100.64.0.5` |
| `controller_hostname` / `controller_vpn_ip` | Edge identity |
| `ztp_dispatch_path` / `ztp_fetch_hits` | From seeking when present |

Routes:

- `GET /api/my-controller/ztp/pending-devices`
- `GET /api/ztp/pending-devices` when controller-edge mode is active (`_ztp_use_controller_edge`)

### 7.2 Recipe options and intent

- Fleet recipe dropdown merges Core catalog with controller-discovered recipes (`ztp.recipes`) when available.
- Operators can record bare-metal intent (`POST /api/fleet/incoming/bare-metal-intent`) with a `recipe_id` that must appear on the allowlist (Core ∪ controller recipes).
- Intents persist under Core data (e.g. `/var/lib/peregrine/ztp/bare_metal_intents.json` — exact path via env).

### 7.3 Claim eligibility (high level)

`fleet_claim_host` enforces (among other checks):

1. Authenticated user with usable profile identity.  
2. `ip` or `mac` present.  
3. **Not already delivered** — if seeking/config outcome is already `delivered` for that host, claim is refused (“only before delivery”).  
4. Not already claimed (409).  
5. User vs org claim paths (`claim_as`, `org_id`, permission `claim_fleet_hosts`).

Ownership labels for Incoming: `POST /api/fleet/incoming-ownership`.

### 7.4 Hub DHCP refusal

Any attempt to enable classic Core hub DHCP hits **`_ztp_refuse_core_hub_dhcp`** (HTTP 403-style product refusal). Zero-touch config POSTs force `server.enabled=False` on the hub. Site DHCP remains a controller concern.

---

## 8. End-to-end flows

### 8.1 Discovery E2E (lab sketch)

1. Controller ZTP enabled on site NIC/subnet; dnsmasq running (auto-start if `enabled`).  
2. Proxmox (or other) blank PXE VM boots — lab VM **231**, MAC **`bc:24:11:9a:83:01`**.  
3. Lease appears on controller; vendor/user class may show PXEClient / iPXE.  
4. Device or operator hits `/ztp/fetch/...`; seeking updates.  
5. Core Fleet (controller branch + matching user) shows Incoming row via pending-devices.  
6. Operator claims + attaches recipe intent before full delivery.

### 8.2 Advanced GitOps recipe update

**GitLab/GitHub talk to Hayabusa Core, not to the controller.** Full write-up: [`docs/GITOPS-CORE.md`](GITOPS-CORE.md).

1. Engineer pushes to GitLab ZTP and/or IaC project.  
2. Webhook hits **Core** (never the controller): `/api/gitops/webhook/gitlab` (or GitHub).  
3. **Core** fetches the repo with **ephemeral** credentials from bridge `gitops.credentials` (long-lived PAT stays in the controller vault).  
4. Core workspace updated; ZTP trees merged; Core pushes onto the site with `iac.import_sync`.  
5. Next `/ztp/fetch` serves new content; `ztp.recipes` reflects updated ids.

```text
GitLab/GitHub --webhook--> Core --API pull--> GitLab/GitHub
                              |
                              +--iac.import_sync--> Controller
```

### 8.3 Novice local recipe edit

1. Edit files under controller `Ansible/ztp` or `bare-metal-ztp` (or sync from Core DevOps).  
2. `list_recipes` / Fleet options pick up new ids.  
3. No Git webhook required.

---

## 9. GitOps (Advanced) — operator checklist

**Who pulls Git:** Hayabusa **Core** only. See [`docs/GITOPS-CORE.md`](GITOPS-CORE.md) for the Core↔Git webhook/API path.  
Controller-facing checklist: `controller-hotfixes/docs/GITOPS.md`.

Summary:

1. Controller bridge connected to Core.  
2. RBAC: `manage_gitops` (builtin admin includes it).  
3. Create vault secrets on controller (PAT + webhook secret) — **never commit real tokens**.  
4. Configure GitOps UI / Advanced setup: provider, instance URL, IaC repo, **ZTP scripts repo**, refs, path prefixes, vault key names, poll interval.  
5. Point repo webhooks at **Hayabusa Core** (`/api/gitops/webhook/gitlab` or `/github`) — **not** the controller.  
6. Confirm **Core** pull → `iac.import_sync` onto controller; workspace read-only locally while enabled.  
7. Placeholders in Git only: `{{ hayabusa_secret:KEY }}`, `<< SECRET:KEY >>`, etc.

**Self-hosted GitLab lab note:** clone/API host must be the reachable IP/name. If GitLab `external_url` advertises a wrong address, configure the GitOps instance URL to the working host (lab example: `http://192.168.1.241`).

Example lab projects (inventory, not shipped fixtures):

- `swoopingbird/hayabusa-ztp-e2e` — dedicated ZTP scripts  
- `swoopingbird/hayabusa-gitops-proxmox-e2e` — IaC / Proxmox OpenTofu E2E  

---

## 10. Image pins, pull, and recreate

### 10.1 Pins (`.env`)

```bash
PEREGRINE_IMAGE=tracedroute/peregrinev2:09152026.1
CONTROLLER_IMAGE=tracedroute/hayabusa-controller:09152026.2
PEREGRINE_PORT=8086
```

### 10.1.1 Release: digests, SBOM/provenance, CVE gate

| Image | Release script | Hub attestations | CVE gate |
|-------|----------------|------------------|----------|
| Controller | `scripts/release-controller-image.sh TAG` | Buildx `--sbom=true` + `--provenance=mode=max` | `scripts/scan-released-image.sh` after push |
| Core | Bake (`docker commit`) + `scripts/release-peregrine-image.sh TAG` | **No** Buildx SBOM/provenance (commit-bake). Rely on **content digest + CVE gate**. Attesting a `FROM baked` re-push is optional checkbox-only and not pursued by default. | Same CVE gate after push |

CVE gate (Trivy, preferred): fail the release on **fixable** `CRITICAL,HIGH` (`SCAN_ONLY_FIXED=1`). Reports in `dist/*.cves*.txt` + SARIF. Large Core images need ~20GB free disk and `SCAN_TIMEOUT=20m` (default). Optional Scout summary: `SCAN_WITH_SCOUT=1`. Bypass: `SKIP_CVE_SCAN=1`. Install Trivy to `~/.local/bin` if missing.

**Production overlay rule (critical):** Core compose may mount the repo at `/hayabusa` for ops paths. `start-server.sh` must **not** copy host `peregrine-src` over baked `/peregrine` unless `PEREGRINE_DEV_SYNC=1`. Leaving sync always-on makes Hub digests look correct while the UI serves stale host files. Prod: keep `PEREGRINE_DEV_SYNC=0` (default). Lab live-edit: set `PEREGRINE_DEV_SYNC=1` in `.env`. Also disable any host “apply_* overlay” entrypoint hooks on production.

```bash
# Controller (build + push + attest + CVE gate)
./scripts/release-controller-image.sh 09152026.3

# Core (bake first, then push + CVE gate)
PEREGRINE_BAKE_TAG=09152026.2 ./scripts/bake-peregrine-image.sh
./scripts/release-peregrine-image.sh 09152026.2
```

Unset shell overrides before compose so `.env` wins:

```bash
cd /home/swoopingbird/hayabusa
unset PEREGRINE_IMAGE CONTROLLER_IMAGE
docker pull tracedroute/peregrinev2:09152026.1
docker pull tracedroute/hayabusa-controller:09152026.2
docker compose up -d --force-recreate --no-deps peregrine
docker compose -f docker-compose.controller.yml up -d --force-recreate --no-deps hayabusa-controller
```

Compose service name for Core is **`peregrine`** (container name `hayabusa-core`). Controller service/container: **`hayabusa-controller`**.

### 10.2 After recreate — smoke checklist

```bash
# Controller health + bridge
curl -sS http://127.0.0.1:8790/healthz
# Expect: ok, bridge_granted, connection_ready

# Core UI
curl -sS -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8086/login   # 200

# DHCP listening on LAN (when ZTP enabled)
ss -ulnp | grep ':67'

# Bridge registry from Core
docker exec hayabusa-core python3 -c "
import sys; sys.path.insert(0,'/peregrine')
from hayabusa_controller_registry import CONTROLLER_REGISTRY
print(CONTROLLER_REGISTRY.list_connected())
print(CONTROLLER_REGISTRY.rpc('admin@100.64.0.5','ztp.leases',{}))
"

# Fetch sample recipe
curl -sS -o /dev/null -w '%{http_code}\n' \
  http://127.0.0.1:8790/ztp/fetch/generic/dispatch.sh
```

Static/source verification script: `scripts/verify-ztp-controller-flow.sh`.

### 10.3 Volumes to preserve

- Core: `peregrine_data` / auth mounts as composed  
- Controller: `hayabusa_controller_data` → `/var/lib/hayabusa-controller`  

Do **not** commit `secrets/`, `.env`, or `bootstrap.env`.

---

## 11. Security model (ZTP-relevant)

1. **Hub DHCP off** — product hard refusal; site DHCP only on controller.  
2. **`/ztp/fetch` is intentionally public** on the LAN appliance — confine paths; treat content as bootstrapping material, not a place for long-lived secrets. Prefer secret placeholders resolved later via vault/hydrate.  
3. **Controller API CSRF** — session APIs require CSRF (`X-CSRF-Token` / form); exempt prefixes include `/healthz`, `/static/`, `/ztp/fetch/`.  
4. **RBAC** — `approve_ztp` for DHCP start/stop; `manage_gitops` for GitOps; Fleet org claim permission `claim_fleet_hosts`.  
5. **Git credentials** — PAT in controller vault; Core uses short-lived bridge-fetched credentials; SSRF host checks on clone URLs.  
6. **No CORS** on controller (same-origin LAN UI).  
7. Container hardening patterns (`no-new-privileges`, etc.) as composed.

---

## 12. Legacy / dual-path notes

Core still contains **legacy hub ZTP** helpers (`ztp_orchestrator`, hub `/ztp/fetch/...`) for environments **without** a selected controller branch. Production intent for site discovery is:

- Controller edge DHCP + fetch  
- Core Incoming via lease relay  
- Hub DHCP disabled  

When documenting or testing, always state whether a **controller branch is selected**.

---

## 13. Troubleshooting

| Symptom | Likely cause | What to check |
|---------|--------------|---------------|
| Incoming empty | Bridge down or wrong user/controller | `healthz` bridge fields; Core `list_connected()`; login identity vs `admin@…` |
| Bridge unauthorized / stuck | WS URL wrong (mesh/public) | `bootstrap.env` / enrollment → LAN `ws://<core-lan>:8791` |
| No UDP 67 | ZTP not started / `enabled` false / port conflict | `ztp.status`; Core must not bind 67; image ≥ `09152026.2` for auto-start |
| Fetch 404 | Path not in workspace trees | `list_recipes`; GitOps sync; file under `Ansible/ztp` or bare-metal |
| Recipes `source: controller` unexpectedly | GitOps/`ztp_repo` not active | Advanced setup; `gitops.json`; agent pull logs |
| Claim refused | Already delivered / already claimed | Seeking `config_outcome`; ownership API |
| Git pull fails | Wrong GitLab URL / PAT / SSRF block | Instance URL reachable; vault PAT scope; webhook secret |
| Compose uses old image | Shell env overrides `.env` | `unset PEREGRINE_IMAGE CONTROLLER_IMAGE` |

---

## 14. Glossary

| Term | Meaning |
|------|---------|
| **Core / Hayabusa / Peregrine** | Hub product container (`hayabusa-core`) |
| **Controller** | Site appliance container (`hayabusa-controller`) |
| **ZTP edge** | Controller DHCP + fetch + seeking subsystem |
| **Incoming** | Fleet pane of unclaimed / seeking hosts |
| **Seeking** | Host has fetched (or attempted) config and awaits full provisioning |
| **Novice** | Local controller workspace is recipe/IaC authority |
| **Advanced** | GitOps; dedicated ZTP repo + IaC repo |
| **Bridge** | Authenticated WebSocket + RPC between controller and Core |
| **Claim** | Assign Incoming host to user/org before delivery completes |

---

## 15. Primary source map

| Topic | Path |
|-------|------|
| Core pending relay, DHCP refuse, my-controller ZTP | `peregrine-src/run.py` |
| GitOps agent + ZTP repo merge | `peregrine-src/hayabusa_gitops_agent.py` |
| UI path rewrite | `peregrine-src/app/static/js/peregrine-controller-branch.js` |
| Controller edge | `controller-hotfixes/ztp_edge.py`, `app/ztp_edge.py` |
| Bridge RPC | `controller-hotfixes/bridge_rpc.py`, `app/bridge_rpc.py` |
| Lifespan auto-start, HTTP routes | `controller-hotfixes/app/main.py` |
| GitOps store / setup | `controller-hotfixes/gitops_store.py`, setup templates |
| **GitOps on Core (webhooks / who pulls Git)** | **`docs/GITOPS-CORE.md`** |
| GitOps operator doc (controller UI) | `controller-hotfixes/docs/GITOPS.md` |
| Image Nest / Windows images | `docs/IMAGE-NEST-AND-WINDOWS-IMAGES.md`, `controller-hotfixes/app/image_nest.py` |
| Verify script | `scripts/verify-ztp-controller-flow.sh` |
| Image pins | `.env` (`PEREGRINE_IMAGE`, `CONTROLLER_IMAGE`) |

---

## 16. Document control

| Item | Value |
|------|-------|
| Title | Hayabusa ZTP & Controller Architecture |
| Lab image pins referenced | Core `09152026.1`, Controller `09152026.2` |
| Companion | `controller-hotfixes/docs/GITOPS.md`, `docs/IMAGE-NEST-AND-WINDOWS-IMAGES.md` |
| Status | Describes implemented behavior; prefer code citations above if docs and code diverge |

---

*End of document.*
