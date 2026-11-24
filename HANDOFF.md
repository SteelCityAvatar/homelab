# Handoff — session context

Last updated 2026-11-24. Detailed history: `docs/sessions/`.

**This file is transitional.** Everything durable belongs in `spec.md`, `bom.md`,
`ROADMAP.md` or an ADR. Fold items in as they close, and delete this file rather
than let it become a second source of truth.

## Server at a glance

Proxmox VE **9.2.24**, kernel **7.0.14-26-pve**, hostname `steel`.
Web UI `https://192.168.1.169:8006` · SSH `ssh root@192.168.1.169` (desktop key,
2026-11-24 — Claude Code can run commands with `ssh -o BatchMode=yes root@192.168.1.169
"<cmd>"`). Boot + VMs on the 970 EVO Plus (ext4 + LVM-thin). Kernel parameter
`nvme_core.default_ps_max_latency_us=0` (APST off, see incident below).
Last checked 2026-11-24: up 20 days, load 0.04.

## Current work — remote access (ADR 0008)

Log: `docs/sessions/2026-11-24.md`. Notes: `docs/notes/remote-access.md`.

- [x] SSH key pair on the desktop; public key in `authorized_keys` with
      `from="192.168.1.0/24,100.64.0.0/10"`; passwordless login verified
- [x] Tailscale 1.104.7 installed on `steel`; `tailscaled` enabled at boot
- [ ] `tailscale up` → approve in the browser → `tailscale ip -4` ← next
- [ ] Tailscale admin console: **disable key expiry** for `steel`; two-factor on the
      Tailscale login
- [ ] Tailscale on the laptop (and phone); test `ssh root@steel` and
      `https://steel:8006` from off the home network (e.g. phone hotspot)
- [ ] Laptop gets **its own** SSH key, added the same way
- [ ] BIOS: restore power on AC loss ⚠️ current setting unknown — needs on-site access

**When working remotely:** no host reboots, no load test. Container work (GPU passthrough,
`nvidia-smi` inside) is fine. If the host restarts unexpectedly, run `nvidia-smi`
once to recreate `/dev/nvidia*` before starting GPU work in the container.

Later: run the Unique Profile MCP server on `steel` over SSH so desktop and laptop
share one profile (not started).

## Model serving, Phase 2 (the container) — paused for remote access

Plan: `docs/plans/model-serving.md`. Container log: `docs/sessions/2026-11-14.md`.

- [x] **Host driver done** (plan Phase 1): NVIDIA 595.104.02, open kernel modules,
      `.run` + DKMS, verified 2026-11-11 (`docs/sessions/2026-11-11.md`). The installer
      is kept at `/root/NVIDIA-Linux-x86_64-595.104.02.run` for the container
- [x] Container **200 `model-serving`** created and started 2026-11-14: Debian 13,
      unprivileged, nesting, 4 cores, 8 GB, 100 GB thin on `local-lvm`, DHCP.
      Enter with `pct enter 200` (no root password set)
- [x] Container on the LAN: `192.168.1.164/24` via DHCP (2026-11-22). ⚠️ Not
      reserved; reserve it on the router alongside `.169` before Ollama goes live
- [x] Host device files present (2026-11-22): `nvidia0`, `nvidiactl`, `nvidia-uvm`,
      `nvidia-uvm-tools`, mode 666 — but created by the first `nvidia-smi`, **not at
      boot**. Nothing NVIDIA in `/etc/modules-load.d/` or `/etc/udev/rules.d/`
- [ ] Boot trigger: `/etc/modules-load.d/nvidia.conf` (`nvidia`, `nvidia_uvm`) + udev
      rule. ⚠️ Whether the modules-load file was written is unconfirmed — check with
      `cat /etc/modules-load.d/nvidia.conf`. Verifying needs a reboot → **on site only**
- [ ] **Pass the GPU into container 200**; install the driver's user-space half
      (`--no-kernel-module`); `nvidia-smi` inside
- [ ] **PSU load test inside the LXC** (~30 min CUDA stress) — gates Ollama
- [ ] Ollama → desktop endpoint → gateway

DKMS built the module for 7.0.14-26 only; the 7.0.2-6 fallback kernel boots
without the NVIDIA module (recovery only). Run long installs over SSH inside `tmux`.
⚠️ An idle tmux session `nvidia` may still be running on the host; harmless, close
with `tmux kill-session -t nvidia`.

## Storage — IronWolfs kept

- **The 2x 4TB IronWolfs are kept** (2026-11-22). The 2026-11-01 plan to return
  them lapsed with the return window. No pool exists on them yet.
- **Decided 2026-11-22:** they become the Vault mirror (~4TB usable). 2x 14TB recerts
  (Toshiba MG07 favored) are **not ordered**; a later purchase, in SATA3/4.
- ⚠️ The irreplaceable data has not been measured and may exceed 4TB. Rule: the
  mirror takes data in priority order (what can never be recreated first: personal
  photos and video, documents, code). Anything that doesn't fit remains a **single
  copy** on the desktop's 8TB until the larger pair exists.
- Before data: measure folder sizes on the desktop, burn in the IronWolfs (SMART
  long + full-surface pass), cap ARC.

## Incident — read-only root (2026-11-01): closed

Probable Samsung NVMe APST dropout; APST disabled. **Passed:** 2026-11-09 — up
5 days 23 h, no NVMe/ext4 errors. If it ever recurs: console first, `dmesg` before
rebooting, then reseat the 970, then suspect the PSU. Write-up:
`docs/sessions/2026-11-01.md`.

## Open items

| # | Item | Notes |
|---|---|---|
| 0 | Remote access | Current work, above (ADR 0008) |
| 1 | GPU into container 200, then the PSU load test inside it | Passthrough can be done remotely; the load test and boot-trigger reboot need someone on site |
| 2 | Storage | IronWolfs = Vault mirror (decided 2026-11-22); 14TB pair not ordered. Measure the irreplaceable data on the desktop, then burn in the IronWolfs |
| 3 | DHCP reservations for `.169` (host) and `.164` (CT 200) | On the router — prevents an address conflict; the model endpoint needs `.164` stable |
| 4 | Security basics | Before real data or Home Assistant: check no port forwards + UPnP off; **IPv6 inbound blocked on the router** (CT 200 has a global IPv6 address); strong root password in a manager; web UI 2FA; Proxmox firewall. SSH keys done 2026-11-24 |
| 5 | Home Assistant VM | `docs/plans/home-assistant.md`; no dependencies |
| 6 | Which room, and the cable route | ADR 0005 left both TBD |
| 7 | Physical bay recount | Spec says 4; confirm when installing the 14TB drives |
| 8 | Confirm RAM speed in BIOS | XMP off; default SPD |
| 9 | Characterize the PSU noise heard 2026-10-25 | `bom.md` #23 lists coil whine as a replace trigger |
| 10 | Fan curves in BIOS | `CPU_FAN` + front `SYS_FAN` PWM; rear DC if 3-pin |

## Known tension — the PSU

~13-year-old Corsair TX750M; replacement deferred, not cancelled (`bom.md` #23).
Light-load voltages passed. **Load-test before the GPU does sustained work.** Do not
delete the desktop copy of irreplaceable data until a Vault backup exists off the
server.

## Things not to re-derive

- **One server GPU: the RTX 2070 Super (8GB).** The GTX 1060 stays out (ADR 0007).
  ~7–8B at Q4 is the ceiling for the always-on model.
- NVIDIA driver from NVIDIA's `.run` with DKMS; host and containers run the **exact
  same version** (ADR 0007).
- The orchestrator belongs on the **server**, not the desktop's 4090 (ADR 0003).
- Do **not** move the 4090 to the server. Direct 4090-to-OLED at sub-1ms already
  beats any streaming or passthrough arrangement.
- Mirrors, not raidz (ADR 0002).
- **Never use the bagged modular SATA cables** — wired for a different PSU.
  Factory-attached SATA and PCIe cables only.
- Recertified enterprise drives, not old consumer drives (ADR 0004).
- The PCIe 5.0 drive stays in the desktop; the server's M.2 is PCIe 3.0 (ADR 0006).
- Use `apt full-upgrade`, never plain `apt upgrade`, on Proxmox.

## Unverified material

`infra/models/litellm-config.example.yaml` is a sketch, not checked against current
LiteLLM docs.

Prices stay out of the BOM — they go stale within weeks. Dated figures appear only
where a decision depended on them (ADR 0004; `docs/sessions/2026-10-31.md`).
