# Handoff — session context

Last updated 2026-11-04. Detailed history: `docs/sessions/`.

**This file is transitional.** Everything durable belongs in `spec.md`, `bom.md`,
`ROADMAP.md` or an ADR. Fold items in as they close, and delete this file rather
than let it become a second source of truth.

## Server at a glance

Proxmox VE **9.2.24**, kernel **7.0.14-26-pve**, hostname `steel`.
Web UI `https://192.168.1.169:8006` · SSH `ssh root@192.168.1.169`.
Boot + VMs on the 970 EVO Plus (ext4 + LVM-thin). Kernel parameter
`nvme_core.default_ps_max_latency_us=0` (APST off, see incident below).

## Current work — NVIDIA driver on the host

Plan: `docs/plans/model-serving.md`, Phase 1. Detail: `docs/sessions/2026-11-03.md`.

- [x] Headers + build tools: `proxmox-default-headers`, `proxmox-headers-$(uname -r)`,
      `build-essential`, `dkms`
- [x] nouveau blacklisted, initramfs rebuilt, rebooted — nouveau no longer loads
- [x] Version chosen: **595.104.02** (ADR 0007: `.run` + DKMS, 2070 Super only)
- [ ] ⚠️ Confirm the installer is in `/root` (`ls -lh /root/NVIDIA*`) — download
      may not have run
- [ ] **Run the installer with DKMS** ← next
- [ ] `nvidia-smi` shows the 2070 Super
- [ ] PSU load test (~30 min GPU stress) — gates the model container
- [ ] Model-serving LXC → Ollama → gateway

If the DKMS build fails against kernel 7.0: try a newer driver, or boot the
fallback kernel 7.0.2-6 from the GRUB menu.

## Storage — drives being swapped

- **Decided: replace the 2x 4TB IronWolfs with 2x 14TB recerts** (Toshiba MG07
  favored). ⚠️ Order status not recorded.
- IronWolfs act as a **temporary dry-run mirror**; nothing may live only on them.
  **Wipe, then return by ≈ 2026-11-20.**
- When the 14TB drives arrive: burn in (SMART long + full-surface pass) inside their
  return window, then `zpool replace` one IronWolf at a time → `autoexpand`.

## Incident watch — read-only root (2026-11-01)

Root filesystem went read-only after ~25 h; drive SMART healthy; probable Samsung
NVMe APST dropout. APST disabled. Ran **2 days 55 min clean** before a planned
reboot on 2026-11-03 — treated as fixed, still watching. **If it recurs: console
first, run `dmesg` before rebooting** (a read-only disk can't save its log), then
reseat the 970, then suspect the PSU. Write-up: `docs/sessions/2026-11-01.md`.

## Open items

| # | Item | Notes |
|---|---|---|
| 1 | NVIDIA driver install | Current work, above |
| 2 | Order / burn in the 14TB drives | Return IronWolfs by ≈ 2026-11-20 |
| 3 | DHCP reservation for `.169` | On the router — prevents an address conflict |
| 4 | Security basics | Before real data or Home Assistant: check no port forwards + UPnP off; strong root password in a manager; SSH keys; web UI 2FA; Proxmox firewall |
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
