# Plan: Home Assistant

**Goal:** Home Assistant OS running as an isolated VM on `steel`, starting on boot,
backed up, reachable on the LAN — and unaffected by anything the ML sandbox does
(ROADMAP Phase 3 exit criterion).

**Type:** VM, not LXC. HAOS is a full appliance (its own OS, supervisor, add-ons),
and isolation is the point (ADR 0001).

**Depends on:** nothing. Runs on `local-lvm` (the 970).

Items marked ⚠️ get verified at the time, not assumed.

---

## Phase A — Prepare

- [ ] **RAM budget.** 32GB total, non-ECC. Proposed split:

  | Consumer | RAM |
  |---|---|
  | Proxmox host | ~2 GB |
  | ZFS ARC (cap it) | 4–8 GB |
  | Home Assistant VM | 4 GB |
  | Model-serving LXC | 8 GB |
  | Gateway LXC | 1 GB |
  | Headroom / media / sandbox | rest |

- [ ] Download the HAOS image on the host (`/var/lib/vz/template`) — command in
      `docs/sessions/2026-11-01.md`. Record the version.

## Phase B — Create the VM

- [ ] Create VM, **no install media**:
  - General: name `homeassistant`, ⚠️ pick a VMID (e.g. `100`), **Start at boot** on
  - System: machine **q35**, BIOS **OVMF (UEFI)**, EFI disk on `local-lvm`,
    **pre-enrolled keys unchecked** (Secure Boot keys block HAOS from booting)
  - Disks: **delete the default disk**
  - CPU: 2 cores, type `host`
  - Memory: 4096 MB, ballooning off
  - Network: default bridge `vmbr0`
- [ ] Import the image: `qm importdisk <VMID> <haos>.qcow2 local-lvm`
- [ ] Hardware → attach the **Unused Disk** (SCSI, Discard on — it's an SSD)
- [ ] Options → boot order: **only** the imported disk
- [ ] ⚠️ Resize the disk to ~32 GB (the image ships small; HA grows with history)

## Phase C — First boot

- [ ] Start the VM; watch its **Console** until it prints the web address
- [ ] Open `http://<VM IP>:8123` → onboarding: owner account, home location, units
- [ ] **DHCP reservation** for the VM's IP on the router (same as `.169` for the host)
- [ ] Record VMID, IP, HAOS version in `infra/homeassistant/README.md`

## Phase D — Make it durable

- [ ] HA's own backups: Settings → System → Backups, on a schedule
- [ ] Proxmox backup of the whole VM (`vzdump`) on a schedule.
      ⚠️ Target storage: `local` (the 970) for now — **same disk as the VM, so it's
      not a real backup** until the pool / an off-server target exists
- [ ] Confirm **Start at boot**: reboot `steel`, check HA comes back by itself

## Phase E — Use it

- [ ] Let auto-discovery find network devices (TV, AVR, Chromecasts, etc.)
- [ ] ⚠️ Zigbee / Z-Wave / Thread later: needs a USB radio passed through to the VM
- [ ] Remote access: **Tailscale add-on** — never port-forward `:8123`

## Phase F — Later (ROADMAP Phase 3)

- [ ] Connect HA's conversation / assist to the LiteLLM gateway
- [ ] **Isolation test:** deliberately break the ML container; HA must keep working

## Done when

HA runs as a VM, restarts by itself after a host reboot, is reachable at a
reserved IP, has scheduled backups, and survives a sandbox blowup.

## Risks

| Risk | Mitigation |
|---|---|
| Secure Boot keys block boot | Uncheck pre-enrolled keys at creation, or disable Secure Boot in the VM's UEFI menu |
| VM IP changes | DHCP reservation |
| Backups on the same disk | Treat as convenience until the pool / off-server target exists |
