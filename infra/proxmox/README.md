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
| SSH | `ssh root@192.168.1.169` — works when the web UI doesn't |
| Kernel params | `nvme_core.default_ps_max_latency_us=0` — disables Samsung NVMe APST deep sleep after a read-only-root incident (2026-11-01, `docs/sessions/2026-11-01.md`) |

## Post-install tasks

- [x] Log in to the web UI from the desktop — 2026-10-29
- [ ] Reserve 192.168.1.169 in the router (DHCP reservation)

- [x] Enterprise repos off, no-subscription on, `apt full-upgrade` — 2026-10-31
- [ ] **Cap ZFS ARC.** Total RAM is 32GB and it is the real ceiling on this
      build. Left uncapped, ARC will starve the HA VM and model containers.
- [ ] NVIDIA driver on the host — in progress: headers, `build-essential`, `dkms`
      installed; nouveau blacklisted (`/etc/modprobe.d/blacklist-nouveau.conf`).
      595.104.02 next (ADR 0007)
- [ ] Create the ZFS mirror (IronWolf dry run → 14TB via `zpool replace`)
- [ ] ~~Configure HDD spindown if living-room placement stands~~ — server is going
      elsewhere (ADR 0005). Revisit only if noise matters in the chosen room
- [ ] Configure UPS integration and clean shutdown
