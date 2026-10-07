# Image Nest & Windows images

**Audience:** operators and integrators who bank Windows (or Linux) install media on a Hayabusa Controller and consume it from OpenTofu / Ansible / bare-metal ZTP.  
**Scope:** how Image Nest creates artifacts, what each deploy target actually banks, which platforms those artifacts fit, and the alternate creation paths.  
**Lab pins (as of 2026-09-22):** Core `tracedroute/peregrinev2:09222026.1`, Controller `tracedroute/hayabusa-controller:09222026.1`.

**Related:** `docs/ZTP-AND-CONTROLLER-ARCHITECTURE.md`, `controller-hotfixes/docs/GITOPS.md`, `devops_iac_defaults/OpenTofu/image-nest/README.tf`, `scripts/e2e-lab-iac/OpenTofu/e2e-windows11-minimal/README.md`.

This document describes **current intended product behavior** as implemented in `controller-hotfixes/app/image_nest.py` (and the Image Nest UI). Prefer code if docs and code diverge.

---

## 1. One-sentence model

**You bring a legal Windows (or Linux) ISO.** The **controller Image Nest** banks a deploy-ready artifact (bootable ISO and/or `install.wim`, optionally an empty qcow2 stub). **Hayabusa Core** runs OpenTofu/Ansible after **LAN Approve** hydrates `<<IMAGE_NEST:…>>` the same way it hydrates secrets. The controller **never** installs or runs OpenTofu.

```
  Your ISO (BYO — Hayabusa does not redistribute Microsoft media)
           │ upload / CLI ingest
           ▼
┌──────────────────────────────────────────────────────────┐
│  Hayabusa Controller — Image Nest                        │
│  iso-cache/ · image-bank/ · playbook-refs/ · registry    │
│  Bake: bare_metal → .iso · hypervisor (Win) → .wim       │
└────────────────────────────▲─────────────────────────────┘
                             │ <<IMAGE_NEST:alias>> hydrate
┌────────────────────────────┴─────────────────────────────┐
│  Hayabusa Core — OpenTofu / Ansible (LAN Approve)        │
│  Proxmox/KVM VM create · PXE/USB bare-metal recipes      │
└──────────────────────────────────────────────────────────┘
```

---

## 2. Is it usable on any platform, or just Proxmox?

**Short answer:** Nest now exposes **five deploy targets** on the Image Nest page for **Windows and Linux** ISOs. Artifacts are still **format-specific**.

| Deploy target (UI) | Windows | Linux | Consumer |
|--------------------|---------|-------|----------|
| **Bare metal** | Bootable **ISO** | Bootable **ISO** | PXE / USB / ZTP |
| **Hypervisor media** | **`install.wim`** (+ empty qcow2 stub) | Banked **ISO** | OpenTofu / KVM attach |
| **Bootable disk** | virt-install → **qcow2** (needs `virt-install` + KVM) | **ISO + qcow2 stub** | Proxmox / KVM guest disk |
| **Hyper-V** | Bootable convert → **VHDX** | **ISO + empty VHDX stub** | Hyper-V |
| **VMware** | Bootable convert → **VMDK** | **ISO + empty VMDK stub** | VMware |

**Normalization:** `proxmox` / `kvm` / `qemu` / `openstack` → hypervisor media. `vhdx` / `hyperv` → Hyper-V. `vmdk` / `esxi` → VMware. `bootable` / `qcow2` → bootable disk.

Bank rows also offer **→ VHDX / → VMDK / → qcow2** convert when a disk already exists (`qemu-img convert`).

**Publish ISO as-is** banks the uploaded medium for bare metal without a strip bake.

---

## 3. What each bake actually produces

UI copy matches this: bare metal → ISO; hypervisor Windows → `install.wim`.

| Deploy target | Windows output | Linux output | Typical use |
|---------------|----------------|--------------|-------------|
| **bare_metal** | Bootable **`.iso`** (`full` = banked copy; `stripped` = WIM strip + xorriso rebuild) | Banked / stripped **ISO** | PXE, USB, optical, bare-metal ZTP |
| **hypervisor** | **`install.wim`** (+ optional **empty** `qemu-img` qcow2 stub) | Banked **ISO** (not WIM) | Feed OpenTofu/virt; not a ready-to-boot Windows disk by default |

### Bake modes (`tool_status`)

| Mode | When controller has | Behavior |
|------|---------------------|----------|
| `offline_wim` | `wimlib-imagex` + `7z` | Preferred: extract/strip WIM or rebuild ISO |
| `kvm` | `qemu-img` + `virt-install` | Optional unattended install **into** qcow2 (needs working KVM; often absent on appliance hosts) |
| `disk_stub` | `qemu-img` only | Empty qcow2 stub alongside WIM |
| `catalog_only` | Missing tools | Manifest/catalog only — not deployable media |

**Critical:** an empty qcow2 stub is **not** installable Windows. Status strings like `ready_wim+disk_stub` mean “WIM banked; disk file is a blank container.” A **bootable** Windows qcow2 requires the optional KVM `virt-install` path (§5) or a separately prepared lab artifact (§6).

Disk: WIM work typically needs **≥ ~9 GB free** on the controller Image Nest volume.

---

## 4. Operator path (UI) — create Windows media

1. Open Controller → **Image Nest** (`GET /image-nest`). RBAC: `read_image_nest` / `manage_image_nest`.
2. **Upload** a legal Windows 11 / Windows Server ISO (or Linux ISO). Files land under `image-nest/iso-cache/` on that controller.
3. Select the uploaded ISO → choose **deploy target**:
   - **Bare metal** — banks a bootable ISO (hydrate e.g. `<<IMAGE_NEST:latest-win11-bare-metal-stripped>>`).
   - **Hypervisor** — Windows banks `install.wim` (hydrate e.g. `<<IMAGE_NEST:latest-win11-stripped>>` / `latest-win11-hypervisor`).
4. Optionally pick **strip** packages present on that ISO (`full` vs `stripped`).
5. **Build** → watch the job (`POST /api/image-nest/build`, `GET /api/image-nest/jobs/{id}`).
6. Confirm bank row + registry aliases under **Saved operating systems**.
7. In OpenTofu/Ansible commit **placeholders only**, e.g.  
   `source_image = "<<IMAGE_NEST:latest-win11-stripped>>"`  
   On **LAN Approve**, the controller resolves the ref into paths / env / `image-nest.auto.tfvars`.

### Useful aliases (Windows client examples)

| Alias | Typical meaning |
|-------|-----------------|
| `latest-win11-iso` / `latest-win11-bare-metal` | Last banked bare-metal ISO |
| `latest-win11-bare-metal-stripped` | Stripped bare-metal ISO |
| `latest-win11-stripped` / `latest-win11-full` | Hypervisor WIM flavors |
| `latest-win11-hypervisor` / `latest-win11-hypervisor-stripped` | Latest hypervisor bank for Win11 |
| `latest-windows-*` | Last banked Windows of that flavor (client or server) |

Field form: `<<IMAGE_NEST:alias:field>>` (e.g. `:source_iso`, `:image_nest_ref`, `:image_flavor`). Bare refs resolve to the primary path (`source_image` / WIM / ISO / qcow2 as registered).

Server families use `winserver` / `latest-winserver-*` slugs the same way.

---

## 5. Other ways to create / bank Windows media

| Path | When to use | Notes |
|------|-------------|-------|
| **UI bake** (§4) | Day-to-day | Main product path |
| **`publish-bare-metal`** | Bank ISO without WIM strip | API: `POST /api/image-nest/publish-bare-metal` |
| **CLI ingest** | Scripted / offline ISO already on disk | `controller-hotfixes/scripts/ingest-win11-to-image-nest.py` (`--iso-path`, `--flavor full\|stripped`) |
| **CLI fetch** | Attempt Microsoft download | `fetch-win11-iso.py` — often blocked (Sentinel / automated fetch); prefer browser download + `--iso-path` |
| **Optional KVM bake** | Controller has `qemu-img` + `virt-install` + usable `/dev/kvm` | Nest may run `virt-install` against the ISO into the qcow2 → status `ready` (true installed disk). Many controller hosts **lack** KVM |
| **`write_tofu_handoff`** | Emit `.auto.tfvars` snippet for Core | Does **not** run tofu on the controller |
| **Lab bootable qcow2** (§6) | Need a ready-to-boot Proxmox disk without controller KVM | Separate prep + bank/serve; not default `offline_wim` |

Autounattend profiles under Nest (`profiles/*_unattend.xml`) are **placeholders**. Guest unattend used in lab E2E lives with Ansible (`scripts/e2e-lab-iac/Ansible/e2e-windows11/files/Autounattend.xml`) and/or offline injection into a prepared disk — Nest WIM bake does **not** inject that unattend by itself.

---

## 6. Lab: bootable Windows qcow2 → Proxmox (proven path)

The E2E stack `scripts/e2e-lab-iac/OpenTofu/e2e-windows11-minimal/` creates a Proxmox VM by **downloading a bootable qcow2 from the controller over HTTP at apply time** — no ISO staged on Proxmox first.

| Piece | Lab value |
|-------|-----------|
| Nest alias | `<<IMAGE_NEST:latest-win11-hypervisor>>` |
| Example banked file | `image-nest/image-bank/win11-hypervisor-bootable-20260922.qcow2` |
| Hydrate | Controller resolves nest ref on LAN Approve; OpenTofu imports disk into the VM |
| Initial config | Unattend in the image (`HAYA-E2E-W11`, local admin from Autounattend) |

This is a **consumption** pattern for Proxmox/KVM. It is **not** the same as the default Nest UI bake (which banks WIM + empty stub). To repeat it you need a **bootable** qcow2 in the bank (KVM bake on controller, or offline prep then register/serve).

---

## 7. OpenTofu / Ansible consumption

1. Prefer committed placeholders: `<<IMAGE_NEST:…>>` and `{{ hayabusa_secret:… }}` (never real controller paths or passwords in git).
2. Queue the job from Hayabusa; on **LAN Approve** the controller hydrates nest refs (parallel to secrets — see GitOps doc).
3. Core runs OpenTofu/Ansible with resolved paths or env vars such as `HAYABUSA_IMAGE_NEST_*`.
4. Stub variables: `controller-hotfixes/devops_iac_defaults/OpenTofu/image-nest/README.tf`.

Example:

```hcl
source_image   = "<<IMAGE_NEST:latest-win11-stripped>>"
image_nest_ref = "<<IMAGE_NEST:latest-win11-stripped:image_nest_ref>>"
image_flavor   = "<<IMAGE_NEST:latest-win11-stripped:image_flavor>>"
```

Ansible-style:

```yaml
win11_image: "{{ lookup('env', 'HAYABUSA_IMAGE_NEST_LATEST_WIN11_STRIPPED') }}"
```

---

## 8. Licensing & ops constraints

- **BYO ISO only.** Hayabusa never distributes Microsoft install media.
- Automated Microsoft fetch may fail; download in a browser and pass `--iso-path`.
- Keep enough free disk on the controller for ISO cache + WIM extract + banked artifacts.
- Controller tooling for rich bake: `wimlib-imagex`, `7z`/`7zz`, `xorriso`; optional `qemu-img`, `virt-install`.
- OpenTofu stays on **Core**, not the controller.

---

## 9. Remaining gaps

- Linux **unattended** virt-install (kickstart / autoinstall) — Linux disk targets bank ISO + empty platform disk stub today
- Full Windows **bare-metal PXE ZTP** recipe pack beyond Nest ISO banking
- Nest Autounattend UI applies to Windows bootable/Hyper-V/VMware bakes; Linux guest config not yet equivalent

---

## 10. Primary API surface (controller)

| Action | Route (sketch) |
|--------|----------------|
| UI | `GET /image-nest` |
| Status / bank | `GET /api/image-nest/status`, `…/bank` |
| Upload | `POST /api/image-nest/upload` |
| Strip options / packages | `…/strip-options`, `…/isos/{id}/packages` |
| Publish bare metal | `POST /api/image-nest/publish-bare-metal` |
| Build / jobs | `POST /api/image-nest/build` (body: `deploy_target`, `unattend`, `disk_gb`, …), `GET …/jobs/{id}` |
| Convert disk | `POST /api/image-nest/bank/{id}/convert` (`format`: `vhdx` \| `vmdk` \| `qcow2`) |
| Registry | `GET /api/image-nest/registry`, `…/registry/{ref}` |
| Tofu handoff | `…/bank/{id}/tofu-handoff` |

---

## 11. Primary source map

| Topic | Path |
|-------|------|
| Nest store / bake | `controller-hotfixes/app/image_nest.py` |
| Linux bake helpers | `controller-hotfixes/app/image_nest_linux.py` |
| HTTP routes | `controller-hotfixes/app/main.py` |
| UI | `controller-hotfixes/app/templates/image_nest.html`, `…/static/js/image-nest.js` |
| Hydrate / bridge | `controller-hotfixes/bridge_rpc.py` (`IMAGE_NEST_*`) |
| Core placeholder sub | `peregrine-src/run.py` |
| Fetch / ingest CLI | `controller-hotfixes/scripts/fetch-win11-iso.py`, `ingest-win11-to-image-nest.py` |
| OpenTofu stub | `controller-hotfixes/devops_iac_defaults/OpenTofu/image-nest/README.tf` |
| Proxmox E2E | `scripts/e2e-lab-iac/OpenTofu/e2e-windows11-minimal/` |
| Lab Autounattend | `scripts/e2e-lab-iac/Ansible/e2e-windows11/files/Autounattend.xml` |

---

## 12. Document control

| Item | Value |
|------|-------|
| Title | Image Nest & Windows images |
| Lab image pins referenced | Core / Controller `09222026.1` |
| Companions | `docs/ZTP-AND-CONTROLLER-ARCHITECTURE.md`, `controller-hotfixes/docs/GITOPS.md` |
| Status | Describes implemented behavior; prefer code citations if docs and code diverge |

---

*End of document.*
