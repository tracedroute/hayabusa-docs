# GitOps on Hayabusa Core — who talks to GitLab/GitHub

**Audience:** operators and engineers working on **Hayabusa Core** (the hub).  
**Related:** controller operator checklist in `controller-hotfixes/docs/GITOPS.md`; bridge/ZTP context in `docs/ZTP-AND-CONTROLLER-ARCHITECTURE.md`.

## Source of truth (modes)

| Mode | Source of truth | Who pulls Git |
|------|-----------------|---------------|
| **Novice** | Controller local `Ansible/` / `OpenTofu/` workspace | Nobody — no Git pull path |
| **Advanced / GitOps** | GitHub or GitLab repository | **Hayabusa Core only** |

In Advanced mode the **controller is not** the Git client for inbound sync. It stores vault secrets (PAT, webhook secret), GitOps config, and team bindings; Core performs the Git fetch and then pushes materialized files to the site.

## Critical rule: webhooks hit Core, not the controller

Register GitLab/GitHub webhooks on the **Hayabusa Core** public URL:

| Provider | Webhook URL |
|----------|-------------|
| GitLab | `https://<hayabusa-core-host>/api/gitops/webhook/gitlab` |
| GitHub | `https://<hayabusa-core-host>/api/gitops/webhook/github` |

Use the **webhook secret** stored in the **controller vault** (same value configured in Advanced setup / GitOps UI). Do **not** point the repo webhook at the controller’s LAN IP or `:8790`.

Implementation: `peregrine-src/hayabusa_gitops_agent.py` (`/api/gitops/webhook/gitlab`, `/api/gitops/webhook/github`).

## How GitLab (or GitHub) communicates with Core

There are two directions. Both involve **Core ↔ Git**, then **Core → controller**.

```text
                    webhook POST (push events)
  GitLab/GitHub  ------------------------------>  Hayabusa Core
       ^                                            |
       | HTTPS API fetch (clone/pull contents)      |
       +--------------------------------------------+
                                                    |
                                         bridge RPC iac.import_sync
                                                    |
                                                    v
                                              Controller
                                         (site workspace + vault)
```

### 1. Git → Core (notify)

On push, GitLab/GitHub POSTs the webhook to Core. Core verifies the shared secret, then starts a sync job for the bound user/controller config.

### 2. Core → Git (fetch)

Core obtains **ephemeral** credentials over the mesh bridge (`gitops.credentials`). The long-lived PAT/deploy token **never leaves the controller vault** as a standing secret on Core. Core then calls the provider HTTP API / git fetch against the configured instance URL and repo.

### 3. Core → controller (materialize)

Core updates its devops workspace, merges ZTP trees when configured, then pushes `Ansible/` and `OpenTofu/` onto the connected controller with **`iac.import_sync`**. Packaged console defaults on the controller are **never overwritten**.

### Other triggers (still Core pulls Git)

| Trigger | Path |
|---------|------|
| Webhook | Git → Core webhook → Core fetch → `iac.import_sync` |
| Controller **Sync now** | Controller bridge event `gitops.pull` → Core fetch → `iac.import_sync` |
| Poll interval | Core poller → same fetch → `iac.import_sync` |

## What the controller still does with Git (outbound only)

Advanced **publish-owned** (team bindings + linked GitHub/GitLab identity) lets a team member push **only owned paths** from the controller to Git using the **team vault PAT**. That is **controller → Git**, not the inbound SoT sync path.

Inbound SoT sync remains: **Git → Core → controller**.

## Novice vs Advanced switch

- **Advanced → Novice:** controller asks Core (`gitops.push_controller`) to push the current Git-backed tree onto the controller first, then disables GitOps so the **local controller workspace** becomes SoT again.
- **Novice → Advanced:** configure Git + team binding; Core begins pull/webhook sync; local UI edits on the controller workspace become read-only while GitOps is enabled.

## Code map (Core)

| Piece | Location |
|-------|----------|
| Webhook routes + pull agent | `peregrine-src/hayabusa_gitops_agent.py` |
| Bridge handlers `gitops.pull` / `gitops.push_controller` | `peregrine-src/run.py` |
| Controller import RPC | `controller-hotfixes/app/bridge_rpc.py` (`iac.import_sync`) |
| Controller GitOps config / vault keys / Sync now | `controller-hotfixes/app/main.py`, `gitops_store.py` |

## Operator checklist (Core-facing)

1. Controller bridge connected to Core.  
2. Advanced GitOps configured on the controller (provider, base URL, repo, vault PAT + webhook secret keys).  
3. Webhook on the GitLab/GitHub project → **Core** URL above (secret = controller vault webhook value).  
4. Confirm Core can reach the Git host (self-hosted GitLab: use a reachable IP/DNS in GitOps base URL).  
5. After a push or Sync now: Core workspace updates, then controller workspace receives non-default files via `iac.import_sync`.  
6. Playbooks in Git use secret **names** only (`{{ hayabusa_secret:KEY }}`, `<<SECRET:KEY>>`); values stay on the controller.

## Security hardening (ops)

### Webhook secrets (required)

- Core **rejects** webhooks when the vault webhook secret is missing or shorter than **16 characters**.
- GitHub: `X-Hub-Signature-256` HMAC; GitLab: `X-Gitlab-Token` must match exactly.
- Per-source IP rate limit defaults to **30 requests/minute** (`HAYABUSA_GITOPS_WEBHOOK_RATE_PER_MIN`). Prefer also rate-limiting at the reverse proxy.

### Least-privilege PAT / deploy token

Store only in the **controller vault** (never commit, never put on Core as a standing secret):

| Provider | Prefer | Scopes / role |
|----------|--------|---------------|
| GitHub | Fine-grained PAT or deploy key + read contents | Contents: **Read** on the IaC repo only; no org admin |
| GitLab | Project access / deploy token | `read_repository` (inbound sync). Publish-owned needs `write_repository` on that project only |

Do **not** use personal owner tokens with site-wide admin for GitOps sync.

### Ephemeral credentials on Core

- At sync time Core fetches `gitops.credentials` over the bridge, uses the token for git fetch/clone, then **scrubs** the remote URL and env token. Errors/logs redact the PAT.
- Residual risk: brief in-memory presence during pull — acceptable vs storing the PAT on Core.

### Linked Git identity (publish-owned)

- Prefer **OAuth Link** (GitHub/GitLab) on the controller.
- **Declare login** is lab bootstrap only. When OAuth for that provider is configured, declare is **blocked** unless `CONTROLLER_ALLOW_DECLARED_GIT_LINK=1`.

### SSRF / host allowlist

- Clone URLs must match the configured `allowed_host` (controller GitOps config). Unexpected hosts are rejected (`ssrf_blocked`).
