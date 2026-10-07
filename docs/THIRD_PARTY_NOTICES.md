# Third-Party Open Source & Trademark Notices — Hayabusa

This file lists open-source software bundled with or invoked by Hayabusa (SECops,
DevOps / IaC, observability, ZTP, fleet discovery, and related features) and explains how third-party
**vendor names** (network equipment manufacturers) are used in the product.
Hayabusa is proprietary software; open-source components remain under their respective
licenses.

**Last updated:** 2026-09-30  
**Image / build reference:** `tracedroute/peregrinev2:09302026.2` (update this line when you tag a new release)

---

## Summary

| Component | Version (typical) | License | Commercial use | Redistribution in Docker image |
|-----------|-------------------|---------|----------------|------------------------------|
| RustScan | 2.2.3 | MIT | Yes | Yes, with copyright notice |
| Zeek | 9.0.0 | BSD-3-Clause | Yes | Yes, with copyright notice |
| tcpdump / libpcap | distro (tcpdump 4.99.1) | BSD / BSD-like | Yes | Yes, with notices |
| ieee-data (OUI) | distro | several (see package) | Yes | Data-only |
| Metasploit Framework | 6.4.97 (reference image) | Framework gem BSD-3-Clause (`LICENSES/ruby_bundler-metasploit-framework-*-LICENSE`); top-level omnibus `LICENSE` metadata may say `"Unspecified"` while listing bundled deps — retain `/opt/metasploit-framework/LICENSE` + `LICENSES/` | Yes (open-source edition, not Metasploit Pro) | Yes, with copyright & bundled dep notices |
| OWASP ZAP | 2.15.x | Apache-2.0 | Yes | Yes, with NOTICE |
| NetExec (`nxc`) | 1.5.1 | BSD-2-Clause | Yes | Yes, with copyright notice |
| Nuclei | 3.11.1 | MIT | Yes | Yes, with copyright notice |
| Nuclei Templates | upstream HEAD | Mixed (per template) | Varies | Do not assume single license |
| Prowler | installed in image | Apache-2.0 | Yes (tool); cloud APIs have separate ToS | Yes, with NOTICE |
| Trivy | 0.74.0 | Apache-2.0 | Yes | Yes, with NOTICE |
| BloodHound.py | installed in image | MIT | Yes | Yes, with copyright notice |
| Ansible / ansible-core | 2.17.14-1ppa~jammy | GPL-3.0-or-later | Yes (tool) | Yes — source offer required |
| OpenTofu (`tofu`) | 1.12.6 | MPL-2.0 | Yes | Yes, with license file |
| Pulumi CLI | 3.265.0 | Apache-2.0 (OSS CLI; verify install path) | Yes | Yes, with copyright/NOTICE |
| Headscale | 0.29.4 | BSD-3-Clause | Yes | Yes, with copyright notice |
| Tailscale (`tailscale` / `tailscaled`) | OSS client rebuild (e.g. 1.103.0-dev…) | BSD-3-Clause (client) | Yes (client) | Yes, with copyright notice |
| strongSwan (IPsec / `charon`) | 5.9.5-2ubuntu2.8 | GPL-2.0+ | Yes | Yes — source offer required |
| dnsmasq | 2.91-0ubuntu0.22.04.1 | GPL-2.0+ | Yes | Yes — source offer required |
| ClamAV | 0.103.12 (Ubuntu) | GPL-2.0 | Yes | Yes — source offer required |
| FFmpeg | 4.4.2-0ubuntu0.22.04.1 | GPL-2+ (Ubuntu build) | Yes | Yes — source offer required |
| Pixiecore (netboot) | built from source (`danderson/netboot`) | Apache-2.0 (upstream Pixiecore) | Yes | Retain Apache NOTICE/license |
| Apache Guacamole / guacd | 1.5.5 (compose) | Apache-2.0 | Yes | Usually run as upstream container images; include NOTICE/license if redistributed |
| FUXA SCADA/HMI | installed under `/opt/fuxa` | MIT | Yes | Installed inside the Hayabusa app container; include copyright/license notice if redistributed |
| ROS 2 Humble tools | Humble | Apache-2.0 and mixed package licenses | Yes | Include package notices/source as applicable |
| MAVLink / PyMAVLink | installed where drone tooling is enabled | LGPLv3 generator with MIT exception for generated code | Yes | Retain notices; generated code exception supports proprietary/commercial use |
| MAVSDK | installed where drone tooling is enabled | BSD-3-Clause | Yes | Retain copyright/license/disclaimer |
| MAVROS | ROS2 package where installed | BSD / GPLv3 / LGPLv3 triple-license | Yes | Select and comply with the license path appropriate to distribution |
| ArduPilot SITL / DroneKit SITL | via `dronekit-sitl` where enabled | GPL-3.0-or-later / package-specific | Yes, subject to GPL terms | Separate unmodified simulator process; provide source offer/notices |
| PX4 Autopilot / PX4 SITL | Not bundled; external connection supported over MAVLink | BSD-3-Clause | Yes | If installed/bundled later, retain PX4 notices/license |
| QGroundControl | Not bundled; optional external/Guacamole launcher supported | Apache-2.0 or GPLv3 dual license | Yes, subject to selected license and Qt terms | Keep external |
| Mission Planner | Not bundled; external launcher supported | GPL-3.0 | Yes, subject to GPL terms | Keep external/separate |
| dump1090 / rtl-sdr | rtl-sdr 0.6.0-4 where installed | GPL lineage / package-specific | Yes, subject to package terms | Provide package source/notices if redistributed |
| OpenDroneID receiver/core | Not bundled; bridge/API integration supported | Apache-2.0 | Yes | Retain notices if bundled |
| NVIDIA Isaac / Isaac Sim / Isaac ROS / Omniverse | Not bundled | NVIDIA product licenses / EULAs | Remote customer deployment only | Obtain NVIDIA licenses separately |
| OpenPLC Editor | `/opt/openplc-editor` | GPL-2.0 with Beremiz / MatIEC components | Yes, subject to GPL terms | Yes — source offer required |
| arduino-cli (bundled with OpenPLC) | 1.0.3 | GPL-3.0 | Yes | Yes — source offer required |
| PLC protocol Python drivers (`pymodbus`, `asyncua`, `python-snap7`, `pycomm3`, `pyads`, `pymcprotocol`) | pip packages | BSD-3-Clause, LGPL-3.0+, MIT, and package-specific terms | Yes, subject to each license | Include license/source notices; LGPL package source availability required |
| Grafana | 11.2.2 | AGPL-3.0 | Yes, subject to AGPL terms | Upstream container; review AGPL before redistribution / multi-tenant hosting |
| Prometheus | 2.54.1 | Apache-2.0 | Yes | Upstream container; include NOTICE/license if redistributed |
| Prometheus Pushgateway | 1.9.0 | Apache-2.0 | Yes | Upstream container |
| Prometheus Alertmanager | 0.27.0 | Apache-2.0 | Yes | Upstream container |
| InfluxDB | 2.7 | MIT / InfluxDB OSS license terms | Yes | Upstream container |
| nginx | 1.27-alpine | BSD-2-Clause (nginx) | Yes | Upstream container |
| postgres | 16-alpine | PostgreSQL License | Yes | Upstream container (usage DB / Guacamole DB) |
| Telegraf | optional / agent recipe | MIT | Yes | Optional install; include license if bundled |
| Prometheus node_exporter | optional / agent recipe | Apache-2.0 | Yes | Optional install |
| Site-proxy WireGuard (`linuxserver/wireguard`) | pinned digest in `docker-compose.yml` | Image mix; WireGuard tools typically GPL-2 | Yes, subject to image/package terms | Separate compose service `[A]`; not the Headscale product mesh |
| Wazuh (manager, indexer, dashboard) | 4.9.2 | GPLv2 (components) | Yes, subject to GPLv2 | Separate upstream containers; Hayabusa proxies `/wazuh/` |
| Shuffle (SOAR) | pinned GHCR digests | AGPL-3.0 | Yes, subject to AGPL | Separate upstream containers; Hayabusa proxies `/shuffle/` |
| MongoDB (Shuffle dependency) | mongo:4.4 | **SSPL-1.0** | Yes, subject to SSPL | Shuffle DB only; SSPL has service-offering restrictions — counsel for hosted models |
| OpenSearch (Shuffle dependency) | 2.11.1 | Apache-2.0 (typical) | Yes | Separate container |

**Nmap**, **Wireshark**, and **tshark** are **not installed** in Hayabusa SECops images and are **not invoked**. Port discovery uses **RustScan** in greppable mode (`-g`, no nmap handoff). Capture uses **tcpdump** with optional **Zeek** summaries.

**ieee-data** supplies OUI vendor names via `/usr/share/ieee-data/oui.txt`.

This is not legal advice. Consult qualified counsel for your distribution model.

---

## Source packages (Ubuntu)

Typical packages include **RustScan** (binary from GitHub releases, not the Debian package that depends on nmap), `zeek`, `tcpdump`, `libpcap0.8`, `ieee-data`, `ansible` / `ansible-core`, `tofu`, `strongswan`, `headscale`, `tailscale`, `dnsmasq`, and `coreutils` (`timeout`). Obtain matching source with `apt-get source <package>` or from the upstream project for your image tag.

**WireGuard®** is a registered trademark of Jason A. Donenfeld. Hayabusa’s **product mesh ([B])** uses the **Tailscale** open-source client (WireGuard-based protocol) with a self-hosted **Headscale** control server where configured—not the commercial Tailscale coordination service unless you connect it separately. Separately, compose profile **site-proxy mesh ([A])** may run the third-party `linuxserver/wireguard` image (pinned by digest in `docker-compose.yml`); that image typically includes GPL-2 WireGuard userspace/tools. Do not confuse [A] with Tailscale Inc’s proprietary product.

**Terraform®** is a registered trademark of HashiCorp. Hayabusa uses **OpenTofu** for IaC; the DevOps UI may accept the legacy command name `terraform` as an alias where configured, but OpenTofu is the implemented engine.

---

## 1. RustScan

- **Copyright:** RustScan / RustScan contributors  
- **License:** MIT  
- **Homepage:** https://github.com/RustScan/RustScan  
- **In Hayabusa:** `/api/pentesting/*` port scans, SECops vulnerability panel (async jobs), LAN host discovery (port-based)

---

## 2. Zeek

- **Copyright:** Zeek contributors (ICSI / open-source community)  
- **License:** BSD-3-Clause  
- **Homepage:** https://github.com/zeek/zeek  
- **In Hayabusa:** Optional post-capture summaries (`conn.log`, etc.) after tcpdump writes a pcap

---

## 3. ieee-data (OUI / vendor names)

- **Package:** Ubuntu `ieee-data`  
- **Purpose:** IEEE OUI listings for MAC vendor resolution in topology/UI  
- **In Hayabusa:** `/usr/share/ieee-data/oui.txt` (preferred over Wireshark `manuf` when present)  
- **Note:** License varies by upstream IEEE assignment; treat as data-only attribution in your BOM.

---

## 4. tcpdump and libpcap

- **tcpdump:** Lawrence Berkeley National Laboratory / tcpdump.org contributors — BSD-style license.  
- **libpcap:** The Tcpdump Group — BSD-style license.  
- **Homepage:** https://www.tcpdump.org/  
- **In Hayabusa:** SECops packet capture endpoint (`/api/pentesting/wireshark`) — UI label may still say “capture”; backend uses tcpdump.

---

## 5. Metasploit Framework

- **Copyright:** 2006–2026, Rapid7, Inc.  
- **License:** BSD-3-Clause (see `/opt/metasploit-framework/LICENSE` in the container)  
- **Homepage:** https://www.metasploit.com/  
- **Note:** Open-source Metasploit Framework is not Metasploit Pro (commercial subscription).  
- **In Hayabusa:** SECops panel (`msfconsole` via pentest routes)  

---

## 6. OWASP Zed Attack Proxy (ZAP)

- **Copyright:** OWASP contributors  
- **License:** Apache License 2.0 (`/opt/zap/license/ApacheLicense-2.0.txt`)  
- **Homepage:** https://www.zaproxy.org/  
- **In Hayabusa:** SECops web application scanning  

---

## 7. NetExec (`nxc` / `netexec`)

- **Copyright:** NetExec / Pennyw0rth contributors (fork of CrackMapExec lineage)  
- **License:** BSD-2-Clause  
- **Homepage:** https://github.com/Pennyw0rth/NetExec  
- **In Hayabusa:** SECops allowlisted CLI (`/api/devops-iac/run`)  

---

## 8. Nuclei

- **Copyright:** ProjectDiscovery, Inc.  
- **License:** MIT  
- **Homepage:** https://github.com/projectdiscovery/nuclei  
- **In Hayabusa:** SECops allowlisted CLI  

### Nuclei Templates (separate work)

Templates are downloaded at runtime from https://github.com/projectdiscovery/nuclei-templates
and may be licensed per-template (often MIT; not universally). Review template metadata
before redistribution of a template bundle.

---

## 9. Prowler

- **Copyright:** Prowler contributors  
- **License:** Apache License 2.0  
- **Homepage:** https://github.com/prowler-cloud/prowler  
- **In Hayabusa:** SECops allowlisted CLI  
- **Note:** Scanning AWS/Azure/GCP requires valid cloud credentials and provider terms of use.  

---

## 10. Trivy

- **Copyright:** Aqua Security Software Ltd.  
- **License:** Apache License 2.0  
- **Homepage:** https://github.com/aquasecurity/trivy  
- **In Hayabusa:** SECops allowlisted CLI  

---

## 11. BloodHound.py

- **Copyright:** BloodHound.py contributors (e.g. dirkjanm / Fox-IT lineage)  
- **License:** MIT  
- **Homepage:** https://github.com/dirkjanm/BloodHound.py  
- **In Hayabusa:** SECops allowlisted CLI (`bloodhound-python`)  

---

## 12. Ansible (ansible-core)

- **Copyright:** Red Hat, Inc. and Ansible project contributors  
- **License:** GNU General Public License v3.0 or later (GPL-3.0-or-later)  
- **Homepage:** https://github.com/ansible/ansible  
- **In Hayabusa:** DevOps / Craft workspaces — `ansible-playbook`, `ansible-galaxy`, inventory, ZTP and network automation playbooks (`/api/devops-iac/run`, Workstations bulk operations)  
- **Source:** `apt-get source ansible-core` (or the version in your image) and https://github.com/ansible/ansible  

---

## 13. OpenTofu

- **Copyright:** OpenTofu contributors (Linux Foundation)  
- **License:** Mozilla Public License 2.0 (MPL-2.0)  
- **Homepage:** https://opentofu.org/ — https://github.com/opentofu/opentofu  
- **In Hayabusa:** DevOps workspaces under `my-tofu-project/` — `tofu` CLI, state sync, inventory export (`/api/devops-iac/opentofu-*`)  
- **Note:** OpenTofu is an open-source fork lineage of Terraform; it is not HashiCorp Terraform.  

---

## 14. Headscale

- **Copyright:** Headscale contributors (e.g. Juan Font Sanz / juanfont lineage)  
- **License:** BSD-3-Clause  
- **Homepage:** https://github.com/juanfont/headscale  
- **In Hayabusa:** Self-hosted control server for Tailscale-compatible mesh (`headscale` CLI, `/api/headscale-*`, tenant namespaces)  
- **Note:** Headscale implements the control-plane protocol; it is not affiliated with Tailscale Inc.

---

## 15. Tailscale (client — WireGuard-based mesh)

- **Copyright:** Tailscale Inc. and contributors  
- **License:** BSD-3-Clause (see https://github.com/tailscale/tailscale/blob/main/LICENSE)  
- **Homepage:** https://github.com/tailscale/tailscale  
- **In Hayabusa:** `tailscale` / `tailscaled` for mesh connectivity (WireGuard data plane); Hayabusa may start/stop `tailscaled` and run `tailscale` subcommands  
- **Trademark:** **Tailscale®** is a trademark of Tailscale Inc. Hayabusa is not Tailscale Inc. and does not imply endorsement.  
- **Note:** Using Tailscale Inc.'s hosted coordination service (if enabled) is subject to **Tailscale's terms of use** in addition to the open-source client license.

---

## 16. strongSwan (IPsec)

- **Copyright:** strongSwan contributors  
- **License:** GPL-2.0+ (see `/usr/share/doc/strongswan/copyright` in the image)  
- **Homepage:** https://www.strongswan.org/  
- **In Hayabusa:** Site-to-site / remote-access IPsec via `ipsec` / `charon` (`/api/ipsec-*`, strongSwan config under `/etc/ipsec.d` and `/etc/peregrine/ipsec/`)  
- **Source (shipped binary in image):** obtain matching source using any of:
  - **Ubuntu (recommended for this image):** https://packages.ubuntu.com/source/jammy/strongswan — package version **5.9.5-2ubuntu2.6** (Jammy)
  - **Command:** `apt-get source strongswan=5.9.5-2ubuntu2.6` (requires `deb-src` for Jammy in `sources.list`)
  - **Upstream:** https://github.com/strongswan/strongswan (release matching 5.9.x if building from upstream)

---

## 17. dnsmasq

- **Copyright:** Simon Kelley and contributors  
- **License:** GPL-2.0+  
- **Homepage:** http://www.thekelleys.org.uk/dnsmasq/  
- **In Hayabusa:** ZTP / lab DHCP and lease parsing for fleet discovery  

---

## 18. Pixiecore (network boot)

- **Upstream:** https://github.com/danderson/netboot (Pixiecore component)  
- **In Hayabusa:** ProxyDHCP / PXE alongside ZTP (`pixiecore` binary built in image)  
- **License:** Apache License 2.0 (Pixiecore / netboot upstream — confirm tag in your image).  

---

## 18a. Pulumi, ClamAV, and FFmpeg (image inventory refresh)

- **Pulumi CLI** — installed at `/usr/local/bin/pulumi` (reference image: v3.265.0). Open-source CLI is typically Apache-2.0; confirm the license file next to the installed binary for your tag. Homepage: https://www.pulumi.com/ / https://github.com/pulumi/pulumi
- **ClamAV** — antivirus engine/daemon packages in the Core image (reference: 0.103.12). License: GPL-2.0. Homepage: https://www.clamav.net/
- **FFmpeg** — Ubuntu Jammy package used by SDR/media paths (reference: 4.4.2-0ubuntu0.22.04.1). The Ubuntu build enables GPL components; treat redistribution as GPL-2+. Homepage: https://ffmpeg.org/

---

## 19. Observability stack

Hayabusa can run an observability stack through the Hayabusa-authenticated proxy and the
`observability/docker-compose.observability.yml` deployment. The typical services are:

- **Grafana** — dashboards, alerting UI, and per-user / per-organization presentation. Typical image: `grafana/grafana:11.2.2`. License: AGPL-3.0. Homepage: https://grafana.com/grafana/
- **Prometheus** — time-series metrics store and PromQL query engine. Typical image: `prom/prometheus:v2.54.1`. License: Apache-2.0. Homepage: https://prometheus.io/
- **Prometheus Pushgateway** — metrics handoff for short-lived jobs and Hayabusa agents. Typical image: `prom/pushgateway:v1.9.0`. License: Apache-2.0. Homepage: https://github.com/prometheus/pushgateway
- **Prometheus Alertmanager** — alert routing. Typical image: `prom/alertmanager:v0.27.0`. License: Apache-2.0. Homepage: https://prometheus.io/docs/alerting/latest/alertmanager/
- **InfluxDB** — metrics API and per-tenant buckets with configured retention. Typical image: `influxdb:2.7`. License: MIT / InfluxDB OSS license terms for the server version in use. Homepage: https://github.com/influxdata/influxdb
- **nginx** — TLS front for Headscale / related routes. Typical image: `nginx:1.27-alpine`. License: BSD-2-Clause. Homepage: https://nginx.org/
- **PostgreSQL** — usage / support databases where compose enables them. Typical image: `postgres:16-alpine`. License: PostgreSQL License. Homepage: https://www.postgresql.org/
- **Telegraf** — optional agent recipe for host and service metrics. License: MIT. Homepage: https://github.com/influxdata/telegraf
- **Prometheus node_exporter** — optional agent recipe for host metrics. License: Apache-2.0. Homepage: https://github.com/prometheus/node_exporter

**Trademarks:** **Grafana®** is a trademark of Grafana Labs. **Prometheus®** is a trademark of The Linux Foundation. **InfluxDB®** and **Telegraf™** are trademarks of InfluxData, Inc. Hayabusa is not affiliated with or endorsed by those vendors unless separately agreed in writing.

---

## 20. Console, Robotics, and PLC tooling

- **Apache Guacamole / guacd** — browser-based remote console gateway. Typical images: `guacamole/guacamole:1.5.5` and `guacamole/guacd:1.5.5`. License: Apache-2.0. Homepage: https://guacamole.apache.org/
- **FUXA SCADA/HMI** — browser-based SCADA/HMI screen builder used by the Workstations SCADA view. Hayabusa installs and launches FUXA inside the application container instead of using a separate FUXA container. License: MIT for the core project. Homepage: https://github.com/frangoteam/FUXA
- **ROS 2 Humble** — robotics command-line and build tooling installed from ROS package repositories. License varies by package; core ROS 2 packages are generally Apache-2.0. Homepage: https://docs.ros.org/en/humble/
- **MAVLink / PyMAVLink** — lightweight vehicle communication protocol and Python tooling used for drone telemetry/control integrations. Generator is LGPLv3-family; generated code includes an MIT exception. Homepage: https://mavlink.io/
- **MAVSDK** — drone application SDK for MAVLink-compatible systems. License: BSD-3-Clause. Homepage: https://mavsdk.mavlink.io/
- **MAVROS** — MAVLink bridge for ROS/ROS2 topic and service workflows. License: available under BSD, GPLv3, and LGPLv3 terms. Homepage: https://github.com/mavlink/mavros
- **ArduPilot SITL / DroneKit SITL** — Hayabusa can launch ArduPilot Copter SITL through the open-source `dronekit-sitl` package for lab simulation. ArduPilot is GPL-3.0-or-later. Hayabusa does not link against or derive from ArduPilot; it starts an unmodified simulator as a separate process and communicates with it over MAVLink. Distributors must provide ArduPilot/DroneKit SITL license notices and corresponding source access/source offer for those GPL components. This does not require releasing Hayabusa application source merely because Hayabusa controls the separate simulator over MAVLink.
- **PX4 Autopilot / PX4 SITL** — Hayabusa supports connecting to externally provided PX4 SITL or PX4 vehicles over MAVLink/MAVSDK/MAVROS. PX4 is generally BSD-3-Clause; PX4 is not bundled in the current image unless you install it separately. Homepage: https://px4.io/
- **QGroundControl** — Hayabusa can store and open tenant-scoped links to an externally hosted or Guacamole-presented QGroundControl dashboard. QGroundControl is dual-licensed Apache-2.0/GPLv3, but its Qt licensing requirements are not a fit for Hayabusa's bundled distribution model. Hayabusa therefore does not bundle QGroundControl; keep it external if an operator wants to use it.
- **Mission Planner** — Hayabusa can store and open tenant-scoped links to an externally hosted Mission Planner desktop. Mission Planner is GPLv3. Hayabusa does not link it into the application; keep it external/separate unless you are prepared to satisfy GPL distribution obligations for that tool.
- **dump1090 / rtl-sdr** — Hayabusa can read dump1090 aircraft JSON to plot ADS-B aircraft in the Robotics Viewer. Common distro dump1090 packages are GPL lineage. Hayabusa treats dump1090 as a separate receiver process and consumes decoded JSON output. The RTL-SDR packages and libraries remain under their upstream licenses. Provide matching package notices/source where redistribution requires it.
- **OpenDroneID / Remote ID receivers** — Hayabusa exposes a Remote ID ingest API for decoded tracks from OpenDroneID/DroneScanner/OpenSpyglass-style bridge clients. OpenDroneID reference receiver/core projects are Apache-2.0; verify the exact license of any specific desktop/mobile scanner before bundling it.
- **NVIDIA Isaac remote integration** — Hayabusa does **not** bundle, license, sublicense, or claim ownership of NVIDIA Isaac, Isaac Sim, Isaac ROS, Omniverse, CUDA, Jetson, or related NVIDIA software. The Workstations NVIDIA ISAAC view is a tenant-scoped remote connector to an operator-provided NVIDIA deployment. Users must obtain and comply with NVIDIA's current licenses, EULAs, GPU driver terms, model/data terms, subscriptions, and trademarks for their own remote server.
- **OpenPLC Editor** — IEC 61131-3 editor installed under `/opt/openplc-editor` with launcher `/usr/local/bin/openplc-editor`. License: GPL-2.0 for the editor distribution; it includes modified Beremiz components and MatIEC compiler components under their upstream licenses. Homepage: https://github.com/thiagoralves/OpenPLC_Editor
- **arduino-cli** — bundled under OpenPLC’s Arduino toolchain (`…/editor/arduino/bin/arduino-cli-l64`, reference v1.0.3). License: GPL-3.0. Homepage: https://github.com/arduino/arduino-cli
- **PLC protocol drivers** — Hayabusa uses open-source Python packages for authorized PLC reads and writes where installed: `pymodbus` (BSD-3-Clause), `asyncua` (LGPL-3.0+), `python-snap7` (MIT), `pycomm3` (MIT), `pyads` (package-specific upstream terms; commonly treated as MIT upstream), and `pymcprotocol` (MIT). Hayabusa also includes small Omron FINS UDP memory read/write helpers. These libraries do not include vendor engineering software licenses.
- **Vendor protocol names** — Siemens S7, Allen-Bradley / Rockwell EtherNet/IP, Schneider Electric Modbus, Beckhoff ADS / TwinCAT, Mitsubishi MC / SLMP, Omron FINS, and OPC UA are referenced for interoperability. The names are trademarks of their respective owners; Hayabusa is not affiliated with or endorsed by those vendors.
- **NVIDIA trademarks** — NVIDIA, NVIDIA Isaac, Isaac Sim, Isaac ROS, Omniverse, CUDA, Jetson, and related marks are trademarks or registered trademarks of NVIDIA Corporation. Hayabusa is not affiliated with, sponsored by, or endorsed by NVIDIA unless separately agreed in writing.

---

## Trademarks and vendor names (routers, switches, firewalls)

Hayabusa displays **vendor and product names** (e.g. **Cisco**, **Juniper**, **Arista**, **Palo Alto Networks**, **Fortinet**, **HPE / Aruba**, **Ubiquiti**, **MikroTik**, **VMware**, **Proxmox**) only to:

- Identify device types your lab may manage (ZTP recipes, Craft / Configure navigation, fleet hints)  
- Show **IEEE OUI** or **DHCP vendor-class** derived labels (via `ieee-data` and lease metadata) such as “likely Cisco” on a MAC address  
- Organize Ansible playbooks and OpenTofu inventory by platform  

**Networking product names:** **WireGuard®** (Jason A. Donenfeld), **Tailscale®** (Tailscale Inc.), and protocol references in UI/docs describe technology families Hayabusa integrates—not partnership or endorsement.

**These names are trademarks of their respective owners.** Hayabusa Solutions / your distributor is **not affiliated with, sponsored by, or endorsed by** those vendors unless you have a separate written agreement. References are **nominative** (identifying compatible equipment and OS families), not an implication that Hayabusa is a partner product.

Do not use Hayabusa screenshots or documentation in a way that suggests endorsement by any network vendor. Vendor logos are not included in stock Hayabusa UI unless you add them under your own brand guidelines.

**ieee-data** OUI strings (e.g. “Cisco Systems”) come from the IEEE assignment public listing; use is for attribution and device identification only.

---

## GPL — source offer (image + related tools)

Hayabusa **ships** these GPL components as part of the container image (or as bundled binaries inside OpenPLC). We **do not** modify and redistribute their source inside the image; instead we **direct you** to the corresponding source for the exact versions below (standard practice for Debian/Ubuntu-based images). Versions match image tag `tracedroute/peregrinev2:09302026.2` unless noted.

| Component | Version in reference image | License | Where to get source |
|-----------|----------------------------|---------|---------------------|
| **strongSwan** (IPsec) | `5.9.5-2ubuntu2.8` | GPL-2.0+ | [Ubuntu source — strongswan (Jammy)](https://packages.ubuntu.com/source/jammy/strongswan) · [strongswan.org](https://www.strongswan.org/download.html) |
| **ansible-core** | `2.17.14-1ppa~jammy` | GPL-3.0-or-later | [Ansible PPA sources](https://launchpad.net/~ansible/+archive/ubuntu/ansible) · `apt-get source ansible-core` |
| **dnsmasq** | `2.91-0ubuntu0.22.04.1` | GPL-2.0+ | [Ubuntu source — dnsmasq (Jammy)](https://packages.ubuntu.com/source/jammy/dnsmasq) · `apt-get source dnsmasq` |
| **ClamAV** | `0.103.12+dfsg-0ubuntu0.22.04.1` | GPL-2.0 | [Ubuntu source — clamav (Jammy)](https://packages.ubuntu.com/source/jammy/clamav) · https://www.clamav.net/ |
| **FFmpeg** | `7:4.4.2-0ubuntu0.22.04.1` | GPL-2+ (Ubuntu build) | [Ubuntu source — ffmpeg (Jammy)](https://packages.ubuntu.com/source/jammy/ffmpeg) · https://ffmpeg.org/ |
| **rtl-sdr** | `0.6.0-4` | package-specific OSS | `apt-get source rtl-sdr` · upstream Osmocom |
| **OpenPLC Editor / MatIEC** | `/opt/openplc-editor` | GPL-2.0 / GPL family | [OpenPLC Editor](https://github.com/thiagoralves/OpenPLC_Editor) and bundled upstream notices |
| **arduino-cli** (OpenPLC path) | 1.0.3 | GPL-3.0 | https://github.com/arduino/arduino-cli · license file beside binary under OpenPLC |
| **ArduPilot / DroneKit SITL** | where enabled | GPL-3.0-or-later / package-specific | Upstream ArduPilot / DroneKit SITL projects |
| **Site-proxy WireGuard image** | `linuxserver/wireguard@sha256:33c5e4260f5ddf9376fcb6f1ff90c0ccc3c63d2ab469595e33d178ecf8a4c4c6` | WireGuard tools typically GPL-2 | Upstream WireGuard + linuxserver image documentation |

**One-shot commands** (on a Jammy system with `deb-src` enabled):

```bash
apt-get source strongswan=5.9.5-2ubuntu2.8
apt-get source dnsmasq=2.91-0ubuntu0.22.04.1
apt-get source ansible-core=2.17.14-1ppa~jammy   # may require the Ansible PPA deb-src line
apt-get source clamav=0.103.12+dfsg-0ubuntu0.22.04.1
apt-get source ffmpeg=7:4.4.2-0ubuntu0.22.04.1
```

Inside a running Hayabusa container confirm versions with:
`dpkg -l strongswan ansible-core dnsmasq clamav ffmpeg rtl-sdr` and tool `--version` flags.

If you cannot use Ubuntu archives (air-gapped or private registry only), contact your **Hayabusa administrator** or distributor identified on your deployment for a source bundle matching your image tag (`THIRD_PARTY_NOTICES.md` header).

## Ansible Galaxy collections (shipped with ansible-core image packages)

The Core image includes Ansible collections under `/usr/lib/python3/dist-packages/ansible_collections`
(e.g. `cisco.ios`, `cisco.nxos`, `arista.eos`, `community.general`, `amazon.aws`, …).
Each collection retains its upstream license (commonly GPL-3.0-or-later and/or
collection-specific terms). List versions with `ansible-galaxy collection list`
inside the image. Collections are separate works from proprietary Hayabusa code.

## AGPL / SSPL — notices + Corresponding Source (Grafana, Shuffle, MongoDB)

Hayabusa runs **unmodified** upstream container images and proxies them after login
(`/grafana/`, `/shuffle/`). We do **not** ship patched Grafana or Shuffle program
source in this repository. Compose may mount provisioning JSON / data volumes only.

**Commercial use:** AGPL permits charging for a service that *uses* these programs.
AGPL applies to Grafana and Shuffle themselves — not to Hayabusa’s separate
proprietary application — so long as they remain separate works (containers +
HTTP proxy) and you do not create a combined derivative by embedding/linking their
source into Hayabusa.

### Corresponding Source (always available)

| Program | Pin | License | Corresponding Source |
|---------|-----|---------|----------------------|
| **Grafana** | `grafana/grafana:11.2.2` | AGPL-3.0 | https://github.com/grafana/grafana/tree/v11.2.2 · https://www.gnu.org/licenses/agpl-3.0.html |
| **Shuffle backend** | `ghcr.io/shuffle/shuffle-backend@sha256:d4a5d2bf1f956955b68b099ba1c38997e4b257b2518215e0427f433515bea5c8` | AGPL-3.0 | https://github.com/Shuffle/Shuffle · AGPL-3.0 text above |
| **Shuffle frontend** | `ghcr.io/shuffle/shuffle-frontend@sha256:4d700a6f0822cb081822bd2fa6c633080553bdd4313aed2c4bdce75b87e82836` | AGPL-3.0 | https://github.com/Shuffle/Shuffle |
| **MongoDB** (Shuffle private DB) | `mongo:4.4` | SSPL-1.0 | https://github.com/mongodb/mongo · https://www.mongodb.com/licensing/server-side-public-license — **not exposed to users**; internal Docker network only |

In-product pages: `/open-source-notices` (HTML) and `/open-source-notices/download` (this file).
Machine-readable source map: `/open-source-notices/source-offer.json`.
Air-gap / pin mismatch: email support@tracedroute.net with your image/compose tag.
Helper script: `scripts/gpl-source-offer.sh`.

**SSPL note:** MongoDB is Shuffle’s private store only (no published ports, no user
Mongo product). That is not “MongoDB as a Service.” Do not productize Mongo hosting
separately without counsel.

---

## Mozilla Public License 2.0 — OpenTofu (summary)

OpenTofu is licensed under the MPL 2.0. A copy of the license is available at https://mozilla.org/MPL/2.0/ and in the OpenTofu distribution. Modified files must remain under MPL; larger works may combine MPL code with proprietary Hayabusa code per MPL terms.

---

## Apache License 2.0 — required notice (subset)

Licensed under the Apache License, Version 2.0 (the "License"); you may not use these
files except in compliance with the License. You may obtain a copy of the License at:

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the
License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
either express or implied. See the License for the specific language governing permissions
and limitations under the License.

**Applies to:** OWASP ZAP, Prowler, Trivy (and other Apache-2.0 components in the image).

---

## MIT License — required notice (subset)

Permission is hereby granted, free of charge, to any person obtaining a copy of this
software and associated documentation files (the "Software"), to deal in the Software
without restriction, including without limitation the rights to use, copy, modify, merge,
publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons
to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or
substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED.

**Applies to:** Nuclei, RustScan, BloodHound.py, `python-snap7`, `pycomm3`, `pymcprotocol` (and other MIT components).

---

## BSD-2-Clause — required notice (NetExec)

Redistribution and use in source and binary forms, with or without modification, are
permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of
   conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright notice, this list of
   conditions and the following disclaimer in the documentation and/or other materials
   provided with the distribution.

**Applies to:** NetExec (`nxc`).

---

## BSD-3-Clause — required notice (Metasploit Framework)

Redistribution and use in source and binary forms, with or without modification, are
permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of
   conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright notice, this list of
   conditions and the following disclaimer in the documentation and/or other materials
   provided with the distribution.
3. Neither the name of the copyright holder nor the names of its contributors may be used
   to endorse or promote products derived from this software without specific prior
   written permission.

**Applies to:** Metasploit Framework (see `/opt/metasploit-framework/LICENSE`), Zeek, Headscale, Tailscale client (BSD-3 upstream), `pymodbus`.

---

## LGPL — required source availability note

LGPL components may be used commercially, but redistribution requires preserving license notices and providing access to the corresponding source for the LGPL component and any modifications to it.

**Applies to:** `asyncua` for OPC UA client support.

---

## Operational use

Use these tools only on systems and networks you are authorized to test. Open-source
licenses do not grant permission to scan third-party targets without authorization.

---

## Contact

For source code or license texts for components in a Hayabusa distribution, contact
your Hayabusa administrator or the distributor identified on your deployment. Prefer
Ubuntu `apt-get source <package>` for distro-built binaries (`tcpdump`, `zeek`, `libpcap0.8`, etc.).
