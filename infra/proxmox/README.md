# Proxmox host

## Install target

M2_1 NVMe. This slot costs zero SATA ports on the MPG Z390 Gaming Plus — see
`docs/spec.md`.

## Install record

| Item | Value |
|---|---|
| Version | Installed from the 9.2-1 ISO (SHA256 verified); **9.2.24** after the 2026-10-31 upgrade |
| Kernel | 7.0.14-26-pve (fallback 7.0.2-6-pve in GRUB) |
| Installed | 2026-10-29 |
| Target disk | Samsung 970 EVO Plus 1TB (NVMe, M2_1) |
| Filesystem | ext4 + LVM-thin (installer default) |
| IP / gateway | 192.168.1.169/24, gateway 192.168.1.1 |
| Hostname | `steel` (confirmed) |
| Timezone | America/New_York |
| Web UI | `https://192.168.1.169:8006`, user `root` |
| SSH | `ssh root@192.168.1.169` — works when the web UI doesn't. Key auth from the desktop (ed25519, `from=` LAN + Tailscale range), 2026-11-24 |
| Remote access | Tailscale 1.104.7, on the tailnet as `steel` (`tailscaled` enabled at boot), 2026-11-24 — ADR 0008, `docs/runbooks/remote-access.md`. ⚠️ Key expiry not disabled — node key expires ≈ 2027-05-23 |
| Kernel params | `nvme_core.default_ps_max_latency_us=0` — disables Samsung NVMe APST deep sleep after a read-only-root incident (2026-11-01, `docs/sessions/2026-11-01.md`) |

## Guests

IDs: VMs in the 100s, containers in the 200s.

| ID | Type | Name | Resources (limits) | Storage | Network | Created |
|---|---|---|---|---|---|---|
| 200 | LXC, unprivileged, nesting | `model-serving` | 4 cores, 8 GB RAM, 512 MB swap | 100 GB thin, `local-lvm` | `vmbr0`, DHCP → `192.168.1.164` (⚠️ not reserved) | 2026-11-14 |

Template: `debian-13-standard_13.6-1_amd64.tar.zst` on `local`.

## Post-install tasks

- [x] Log in to the web UI from the desktop — 2026-10-29
- [ ] Reserve 192.168.1.169 (host) and 192.168.1.164 (CT 200) in the router
- [x] SSH key login from the desktop — 2026-11-24
- [x] Tailscale installed and `steel` joined the tailnet — 2026-11-24
- [ ] Tailscale: disable key expiry for `steel` — deferred; ⚠️ expires ≈ 2027-05-23
- [ ] BIOS: restore power on AC loss
- [ ] NVIDIA device files at boot — `/etc/modules-load.d/nvidia.conf` +
      `/etc/udev/rules.d/70-nvidia.rules` written 2026-11-24; ⚠️ unproven until an
      on-site reboot

- [x] Enterprise repos off, no-subscription on, `apt full-upgrade` — 2026-10-31
- [ ] **Cap ZFS ARC.** Total RAM is 32GB and it is the real ceiling on this
      build. Left uncapped, ARC will starve the HA VM and model containers.
- [x] NVIDIA driver on the host — 595.104.02 via `.run` + DKMS (ADR 0007), 2026-11-11.
      nouveau blacklisted (`/etc/modprobe.d/blacklist-nouveau.conf`). `tmux` installed
- [ ] Create the ZFS mirror on the IronWolfs (kept, 2026-11-22)
- [ ] ~~Configure HDD spindown if living-room placement stands~~ — server is going
      elsewhere (ADR 0005). Revisit only if noise matters in the chosen room
- [ ] Configure UPS integration and clean shutdown
