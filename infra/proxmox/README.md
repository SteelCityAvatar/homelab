# Proxmox host

## Install target

M2_1 NVMe. This slot costs zero SATA ports on the MPG Z390 Gaming Plus — see
`docs/spec.md`.

## Install record

| Item | Value |
|---|---|
| Version | Proxmox VE 9.2-1 ISO (SHA256 verified against the official `SHA256SUMS`); reports 9.2.2 after install |
| Installed | 2026-10-29 |
| Target disk | Samsung 970 EVO Plus 1TB (NVMe, M2_1) |
| Filesystem | ext4 + LVM-thin (installer default) |
| IP / gateway | 192.168.1.169/24, gateway 192.168.1.1 |
| Hostname | `steel` (confirmed) |
| Timezone | America/New_York |
| Web UI | `https://192.168.1.169:8006`, user `root` |

## Post-install tasks

- [x] Log in to the web UI from the desktop — 2026-10-29
- [ ] Reserve 192.168.1.169 in the router (DHCP reservation)

- [ ] Remove the enterprise repo / add the no-subscription repo, then update
- [ ] **Cap ZFS ARC.** Total RAM is 32GB and it is the real ceiling on this
      build. Left uncapped, ARC will starve the HA VM and model containers.
- [ ] Install the NVIDIA driver on the host (required for LXC passthrough)
- [ ] Create ZFS pools: Vault (mirror) and Bulk
- [ ] ~~Configure HDD spindown if living-room placement stands~~ — server is going
      elsewhere (ADR 0005). Revisit only if noise matters in the chosen room
- [ ] Configure UPS integration and clean shutdown
