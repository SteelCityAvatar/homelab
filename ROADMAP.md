# Roadmap

Phases are gated. Do not start a phase until its blocker clears.

---

## Phase 0 — Verify hardware

**Blocker for everything.** No purchases until this completes.

- [x] Bench-test the board for POST (`docs/runbooks/bench-test-motherboard.md`)
      — passed 2026-10-05. Board, CPU, RAM confirmed working. Tested with one
      DIMM only; Both sticks since installed and passed MemTest86 (2026-10-27).
- [x] Count 3.5" bays in the HAF XB EVO — 4, per manufacturer spec (2.5" mounts are conversions of the same 4). ⚠️ Physical recount not done — confirm when installing drives
- [x] Record RAM config — 2x16GB DDR4-3200 matched kit, DIMMA2 + DIMMB2. XMP off — runs at the kit's default SPD speed (⚠️ likely 2133–2666, confirm in BIOS). Separate 16GB kit kept as spare (not mixed in). CL/voltage TBD (irrelevant with XMP off)
- [x] MemTest86 on the 2x16 kit — **passed 2026-10-27**, both sticks installed.
      CPU 38°C during the run
- [x] Record PSU model and age — Corsair TX750M, bought new ~2013 (~13 yrs). ⚠️ Exceeds the 8-yr replace threshold in bom.md. Replacement **deferred** 2026-10-05 — conditions in bom.md #23
- [x] Verify the PSU and modular cables with a multimeter — 2026-10-25. Voltages in
      spec at light load. Bagged SATA cables **BAD**; factory SATA cable verified
      (`docs/runbooks/verify-modular-psu-cables.md`)
- [x] Decide final placement — outside the living room, wired ethernet (ADR 0005). ⚠️ Room and cable route TBD

**If the board is dead:** source a used 300-series replacement. See
`docs/spec.md` contingency section. The project continues.

---

## Phase 1 — Storage foundation  ← CURRENT

**Data drives:** swapping the 2x 4TB IronWolfs for **2x 14TB recertified** drives
(decided 2026-11-01). The IronWolfs are a temporary dry-run mirror, returned by
≈ 2026-11-20. Boot + hot tier on the 970 (ADR 0006).

- [x] Confirm the 970 Evo Plus capacity — **1TB**, covers boot + hot tier (2026-10-24)
- [ ] Order the 2x 14TB recerts
- [ ] Burn in the 14TB drives **inside their return window** — SMART long test,
      then a full-surface write/read pass. A drive that dies in week one goes back
      to the seller instead of through RMA
- [ ] Wipe and return the IronWolfs by ≈ 2026-11-20
- [x] SMART check on the 970 — healthy (2026-11-01: 0 media errors, 1% used)
- [x] Swap NVMe drives between machines (ADR 0006) — 970 now in the server.
      Desktop confirmed stable on the 9100 Pro (2026-10-27) and it was a clone,
      so the 970 holds nothing unique — safe for Proxmox to wipe
- [x] Fit the 120mm fan to the existing heatsink (bom.md #12) — 2026-10-24
- [ ] Assemble — mostly done 2026-10-25: IronWolfs (SATA1/2, factory SATA power),
      GPU (factory PCIe power) and front fans in. Ethernet in (borrowed desktop Cat6) 2026-10-29. Remaining: fan curves
- [x] Install Proxmox VE to M2_1 (970 Evo Plus) — **9.2, 2026-10-29.** ext4,
      192.168.1.169, web UI on :8006. Hostname `steel`
- [x] Log in to the web UI from the desktop — 2026-10-29 (VE 9.2.2)
- [ ] Reserve 192.168.1.169 for the server in the router (DHCP reservation)
- [x] Switch to the no-subscription repository and update — 2026-10-31, VE 9.2.24,
      rebooted onto kernel 7.0.14-26
- [ ] Create the mirror (IronWolfs as a dry run → `zpool replace` with the 14TBs)
- [ ] Cap ZFS ARC (32GB total RAM)
- [ ] Migrate data from the desktop's 8TB onto the 14TB mirror
- [ ] Set up a Vault backup **off the server** — bom.md #23 condition (a) blocks
      deleting the desktop copies until this exists
- [ ] Later: a second mirror pair as needed (2 bays + 2 SATA ports free)

**Exit criteria:** data is on the server, mirrored where it matters, backed up.

---

## Phase 2 — Model serving

Detailed plan: `docs/plans/model-serving.md`.

- [ ] NVIDIA driver on the Proxmox host — **in progress**: headers + build tools in,
      nouveau blacklisted; 595.104.02 `.run` with DKMS next (ADR 0007)
- [ ] PSU load test under sustained GPU load (`bom.md` #23 c)
- [ ] LXC with GPU device passthrough
- [ ] Ollama (or llama.cpp) on the 2070 Super
- [ ] Ollama on the desktop, bound to LAN
- [ ] LiteLLM gateway with health-check + fallback chain
- [ ] Verify graceful degradation: launch a game, confirm the router stops
      escalating rather than hanging

**Exit criteria:** a request routes locally, escalates to the 4090 when free,
and falls back cleanly when it is not.

---

## Phase 3 — Home automation

Detailed plan: `docs/plans/home-assistant.md`. Can start before Phase 2 — it
depends on nothing.

- [ ] Home Assistant OS in a VM
- [ ] Integrate with the LiteLLM gateway
- [ ] Verify HA survives a deliberate sandbox blowup

**Exit criteria:** breaking the ML container does not affect the house.

---

## Phase 4 — Polish and expand

- [ ] Media server (Jellyfin or Plex — TBD)
- [ ] UPS + clean shutdown on power loss
- [ ] 2.5GbE direct link between nodes
- [ ] ~~Noise mitigation if living-room placement stands~~ — dropped, see ADR 0005
- [ ] Snapshot/rollback workflow documented for the sandbox
- [ ] Add second HDD pair as budget allows
