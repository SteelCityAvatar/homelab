# Proxmox host

## Install target

M2_1 NVMe. This slot costs zero SATA ports on the MPG Z390 Gaming Plus — see
`docs/spec.md`.

## Post-install tasks

- [ ] Remove the enterprise repo / add the no-subscription repo
- [ ] **Cap ZFS ARC.** Total RAM is 32GB and it is the real ceiling on this
      build. Left uncapped, ARC will starve the HA VM and model containers.
- [ ] Install the NVIDIA driver on the host (required for LXC passthrough)
- [ ] Create ZFS pools: Vault (mirror) and Bulk
- [ ] Configure HDD spindown if living-room placement stands
- [ ] Configure UPS integration and clean shutdown
