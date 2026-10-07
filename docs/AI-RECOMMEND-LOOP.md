# AI recommend loop (Core + Controller)

**Ship shape:** model select in AI helper → Core Prometheus telemetry (`device_id`, `controller_branch`) + Controller OpenTofu `iac.state_snapshot` → `recommendation.v1` → **Controller LAN Approve**.

API keys remain on the Controller vault (secret names only on Core). Prefer **Profile → AI keys** so alert cards can Suggest without re-entering a secret name.

| Piece | Location |
|-------|----------|
| Core recommend module | `peregrine-src/peregrine_ai_recommend.py` |
| Alert→AI gate (opt-in) | `peregrine-src/hayabusa_ai_alert_recommend.py` |
| Alert color families | `peregrine-src/hayabusa_alert_colors.py` |
| Alert store | `peregrine-src/hayabusa_alert_store.py` |
| Schema | `peregrine-src/schemas/recommendation.v1.json` |
| Alerts UI | `peregrine-src/app/static/js/peregrine-alerts-notifier.js` |
| AI UI | `peregrine-src/app/templates/_ai_agent_console.html` |
| Prom rules | `observability/prometheus/alerts/hayabusa.rules.yml` |

### Alert families (colors + friendly labels)

| Family | Palette | Label |
|--------|---------|-------|
| **disaster** | red / orange / yellow / blue | **Emergency** |
| **disaster_recovery** | rose / amber / fuchsia / stone | **Recovery** |
| **infrastructure** | violet / indigo / cyan / slate | **Network** |

### Simple UX
1. Open Alerts → **Network** tab.
2. On an alert with a device id: **Suggest playbook** (uses Profile → AI keys).
3. Review on the card → **Send to Approve**.
4. Optional: **Watch Network alerts** in the panel for background suggestions when alerts fire.

Recovery / Emergency alerts never auto-call the model.

### Core HTTP
- `GET /api/ai/telemetry/snapshot?device_id=&controller_branch=`
- `GET /api/my-controller/iac/state-snapshot?device_id=`
- `POST /api/ai/recommend`
- `POST /api/ai/recommend/from-alert` — one-click from Network alert card
- `POST /api/ai/recommend/approve` → `job.run` pending LAN Approve
- `GET|PUT /api/ai/recommend/alert-opt-in` (`{ watching: true }` auto-uses Profile keys)
- `GET /api/ai/recommend/suggestions`
- `POST /api/ai/recommend/process-alerts`

### Controller → Core metrics
Controllers stamp `controller_branch` + `device_id` and push to Core Pushgateway (`telemetry.push` RPC / `telemetry_push.py`).
