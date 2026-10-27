# Handoff — session context

Last updated 2026-10-26. Detailed log of the 2026-10-22 → 26 sessions:
`docs/sessions/2026-10-25.md`. Rewritten after the repo moved ahead of the planning
conversation that created it.

**This file is transitional.** Everything durable here belongs in `spec.md`,
`bom.md`, `ROADMAP.md` or an ADR. When the open items below close, fold them in
and delete this file rather than letting it become a second source of truth.

## Where the project stands

Phase 0 is mostly done. The board POSTs (2026-10-05), RAM is a 2x16GB kit with
two slots free, placement is settled (ADR 0005), and the PSU question has been
decided-with-conditions rather than left open (bom.md #23).

**Phase 1 no longer needs drive purchases.** 2x 4TB IronWolf plus the desktop's
970 Evo Plus cover the Vault tier and boot. See ADR 0006.

## Immediate open items

| # | Item | Why it matters |
|---|---|---|
| 1 | MemTest86 overnight on the 2x16 kit | **Running** since 2026-10-26. Must finish with 0 errors before ZFS holds real data |
| 2 | ~~970 Evo Plus capacity~~ | **Resolved: 1TB.** Boot + hot tier share M2_1; M2_2 stays free |
| 3 | Burn in the new IronWolfs (bought 2026-10-21) | Do it inside the return window — a week-one failure goes back to the store, not through RMA |
| 4 | Which room, and the cable route | ADR 0005 left both TBD. Apartment — wall runs may need landlord sign-off |
| 5 | Physical bay recount | Spec says 4; owner saw "8 places". Confirm when installing drives |
| 6 | Confirm RAM speed in BIOS | XMP off; running at default SPD |
| 7 | Characterize the PSU noise heard 2026-10-25 | `bom.md` #23 lists coil whine as a replace trigger |
| 8 | Keep vs. return the IronWolfs | Keep if the irreplaceable part of the desktop's 8TB fits in ~3TB usable. Decide inside the return window |
| 9 | Ethernet to the server | It has no network yet |
| 10 | Fan curves in BIOS | `CPU_FAN` + front `SYS_FAN` in PWM mode; rear in DC mode if it's 3-pin |

## Known tension worth tracking

The PSU is a ~13-year-old Corsair TX750M and replacement is deferred, not
cancelled. bom.md #23 carries the conditions. The one that constrains Phase 1:
**do not delete the desktop copy of irreplaceable data until a Vault backup
exists off the server.** With only 4TB usable against ~5TB of data, the desktop
is staying in the loop for a while regardless — that happens to align.

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

No prices appear anywhere on purpose — the 2026 storage market moves fast enough
that written figures go stale within weeks.
