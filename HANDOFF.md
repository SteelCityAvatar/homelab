# Handoff — session context

Last updated 2026-11-01. Detailed history: `docs/sessions/`.

**This file is transitional.** Everything durable here belongs in `spec.md`,
`bom.md`, `ROADMAP.md` or an ADR. When the open items below close, fold them in
and delete this file rather than letting it become a second source of truth.

## Current step (2026-11-01)

**Proxmox VE 9.2.24, fully updated**, kernel **7.0.14-26-pve**, web UI at
`https://192.168.1.169:8006` (hostname `steel`), SSH as `root`.

**Next, one step at a time:**

1. **Decided: swap to 2x 14TB recerts.** IronWolfs stay
   installed as a **temporary dry-run mirror** until the new drives arrive, then get
   wiped and returned before ≈ **2026-11-20**. Nothing irreplaceable lives *only* on
   them. Upgrade path: `zpool replace` each IronWolf with a 14TB drive one at a
   time, then `autoexpand` grows the pool in place — no rebuild, data stays.
   **Building the services meanwhile** (on `local-lvm`, the 970): Home Assistant VM,
   model-serving LXC (needs the host NVIDIA driver first), LiteLLM gateway LXC.
   ⚠️ Order between them TBD — see `docs/plans/`.
2. Burn in whichever drives stay — confirm device names first (steel → Disks, or
   `lsblk -o NAME,SIZE,MODEL`), then SMART long tests, then a full-surface pass.
   SMART check on the 970 too.
3. DHCP reservation for `.169` on the router.
4. Then: Vault mirror + ZFS ARC cap (Phase 1).

## Incident watch (from 2026-11-01)

The root filesystem went **read-only** after ~25 h (probable Samsung NVMe APST
dropout; drive SMART healthy). APST disabled via kernel parameter at 21:10.
**Leave the server running ≥ 3 days** before calling it fixed. If it recurs:
console first (`dmesg` before rebooting!), then reseat the 970, then suspect the PSU.
Detail: `docs/sessions/2026-11-01.md` → "Incident".

## Where the project stands

Phase 0 is done: the board POSTs, MemTest86 passed, placement is settled (ADR
0005), and the PSU is decided-with-conditions (bom.md #23). Phase 1 is underway:
Proxmox installed and updated; data drives being swapped for 2x 14TB.

## Immediate open items

| # | Item | Why it matters |
|---|---|---|
| 1 | ~~MemTest86 on the 2x16 kit~~ | **Passed 2026-10-27** |
| 2 | ~~970 Evo Plus capacity~~ | **Resolved: 1TB.** Boot + hot tier share M2_1; M2_2 stays free |
| 3 | Burn in the 14TB drives on arrival | Inside their return window — a week-one failure goes back to the seller, not through RMA |
| 4 | Which room, and the cable route | ADR 0005 left both TBD |
| 5 | Physical bay recount | Spec says 4; confirm when installing the 14TB drives |
| 6 | Confirm RAM speed in BIOS | XMP off; running at default SPD |
| 7 | Characterize the PSU noise heard 2026-10-25 | `bom.md` #23 lists coil whine as a replace trigger |
| 8 | ~~Keep vs. return the IronWolfs~~ | **Returning** — swap to 2x 14TB. Wipe first, back by ≈ 2026-11-20 |
| 9 | ~~Ethernet to the server~~ | **Connected** — borrowed desktop Cat6, `192.168.1.169`. Permanent run tied to #4 |
| 10 | Fan curves in BIOS | `CPU_FAN` + front `SYS_FAN` in PWM mode; rear in DC mode if it's 3-pin |

## Known tension worth tracking

The PSU is a ~13-year-old Corsair TX750M and replacement is deferred, not
cancelled. bom.md #23 carries the conditions. The one that constrains Phase 1:
**do not delete the desktop copy of irreplaceable data until a Vault backup
exists off the server.**

Re-evaluate the PSU before Phase 2 puts both GPUs under sustained load.

## Things not to re-derive

- The two server GPUs **will not pool**. Mismatched generations (Turing +
  Pascal), mismatched memory. Two independent workers, never one 17GB pool.
- The orchestrator belongs on the **server**, not the desktop's 4090 (ADR 0003).
  The desktop is a Windows gaming/theater box with low availability.
- Do **not** move the 4090 to the server. Direct 4090-to-OLED at sub-1ms already
  beats any streaming or passthrough arrangement.
- Mirrors, not raidz (ADR 0002).
- The server GPU is an **RTX 2070 Super (8GB)**, not a 2080 Ti. ~7-8B at Q4 is the
  ceiling for the always-on model.
- **Never use the bagged modular SATA cables** — they tested wrong for the TX750M
  (2026-10-25). Factory-attached SATA and PCIe cables only.
- A local marketplace lot of 15-year-old consumer drives was evaluated and
  rejected on per-TB cost, age, and bay/port budget (ADR 0004).
- The PCIe 5.0 drive does not go in the server — Z390's M.2 slots are PCIe 3.0,
  and the server is the machine that benefits least from fast NVMe (ADR 0006).

## Unverified material in this repo

`infra/models/litellm-config.example.yaml` is a sketch written from the general
shape of that schema, not from current documentation. Check it against live
LiteLLM docs before running it.

Prices stay out of the BOM — the 2026 storage market moves fast enough that
figures go stale within weeks. Dated figures appear only where a decision depended
on them (ADR 0004; the drive comparison in `docs/sessions/2026-10-31.md`).
