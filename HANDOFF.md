# Handoff — session context

Last updated 2026-11-24. Detailed history: `docs/sessions/`.

**This file is transitional.** Everything durable belongs in `spec.md`, `bom.md`,
`ROADMAP.md` or an ADR. Fold items in as they close, and delete this file rather
than let it become a second source of truth.

## Starting a remote session (no repo checkout, e.g. a chat assistant)

Repo: https://github.com/SteelCityAvatar/homelab — read this file, then
`docs/runbooks/remote-access.md` (connect, safe vs on-site-only, recovery) and
`CLAUDE.md` (constraints and working style: one step at a time, confirm before
anything destructive, mark unverified items ⚠️).

- The operator connects over **Tailscale**; the server is `steel` on the tailnet.
  Web UI `https://steel:8006`; its **>_ Shell** gives a root shell from any browser,
  including a phone.
- An assistant without SSH can't run commands: give one command at a time, wait for
  the pasted output.
- **Keep a command log** in the reply (exact command, read-only vs destructive,
  outcome). It gets copied into `docs/sessions/<date>.md` when back at a repo checkout.
- **No host reboots and no load test remotely** (runbook table).

### Remote setup still to do (can be finished remotely)

1. **Laptop SSH key** (laptop on Tailscale):
   `ssh-keygen -t ed25519 -C "laptop-to-steel"` on the laptop, then add the
   `.pub` line on `steel` through the web UI shell, with the same limit as the
   desktop key:
   `echo 'from="192.168.1.0/24,100.64.0.0/10" <contents of id_ed25519.pub>' >> /root/.ssh/authorized_keys`
   (`>>`, never `>`). Test: `ssh -o BatchMode=yes root@steel hostname`.
2. **Claude Code on the laptop:** clone the repo; it can then run server commands
   over that key, same as the desktop.
3. **Ops container with Claude Code** (optional; container work is remote-safe):
   new unprivileged LXC (e.g. CT 202 `ops`) holding Claude Code, a repo clone and
   `tmux`. Its own SSH key to the host, `from=` its own address only; a GitHub
   **deploy key** scoped to this repo for commits. Not on the host itself: an agent
   there is root on the hypervisor, and Proxmox hosts stay minimal. Approval prompts
   stay on. Reach it by SSH over Tailscale + `tmux attach`.
4. **Unique Profile on `steel`** (optional, later): run the MCP server on the host
   over SSH so desktop and laptop share one profile.
5. Then resume model serving: GPU passthrough into CT 200 is remote-safe; the PSU
   load test and the NVIDIA boot-trigger reboot are on-site only.

## Server at a glance

Proxmox VE **9.2.24**, kernel **7.0.14-26-pve**, hostname `steel`.
Web UI `https://192.168.1.169:8006` · SSH `ssh root@192.168.1.169` (desktop key,
2026-11-24 — Claude Code can run commands with `ssh -o BatchMode=yes root@192.168.1.169
"<cmd>"`). Boot + VMs on the 970 EVO Plus (ext4 + LVM-thin). Kernel parameter
`nvme_core.default_ps_max_latency_us=0` (APST off, see incident below).
Last checked 2026-11-24: up 20 days, load 0.04.

## Current work — remote access (ADR 0008)

Log: `docs/sessions/2026-11-24.md`. Runbook: `docs/runbooks/remote-access.md`.

- [x] SSH key pair on the desktop; public key in `authorized_keys` with
      `from="192.168.1.0/24,100.64.0.0/10"`; passwordless login verified
- [x] Tailscale 1.104.7 installed on `steel`; `tailscaled` enabled at boot
- [x] `steel` joined the tailnet (`tailscale up`, 2026-11-24); `tailscaled` enabled + active
- [ ] Tailscale admin console: **disable key expiry** for `steel`; two-factor on the
      Tailscale login ← next
- [ ] Tailscale on the laptop (and phone); test `https://steel:8006` from off the
      home network (e.g. phone hotspot)
- [ ] Laptop SSH key — steps above; can be done remotely via the web UI shell
- [ ] BIOS: restore power on AC loss ⚠️ current setting unknown — on site only
      (reboot into BIOS)

Remote rules and recovery: `docs/runbooks/remote-access.md`.

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
- [x] Boot trigger written 2026-11-24: `/etc/modules-load.d/nvidia.conf` (`nvidia`,
      `nvidia_uvm`) + `/etc/udev/rules.d/70-nvidia.rules` (`udevadm verify` passed)
- [ ] ⚠️ Prove it: after the next host reboot (**on site only**), `ls -l /dev/nvidia*`
      shows boot-time timestamps without running `nvidia-smi` first
- [ ] CT 200 has no `onboot` — set it once GPU passthrough works
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
