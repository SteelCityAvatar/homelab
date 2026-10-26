# CLAUDE.md

Context for Claude Code working in this repo.

## What this project is

Build and operate a home server ("Node B") that acts as the always-on half of a
two-machine ecosystem. The other half ("Node A") is an existing Windows 11
desktop that doubles as a home theater PC and must remain untouched.

The server's job: bulk storage, local model serving, and home automation — so
that Node A can stay a gaming/theater machine.

## Current status

**Phase 0, in progress.** Motherboard POST verified 2026-10-05 (board, CPU, RAM
working). Phase 0 items recorded. Still open: memtest (unblocked; USB stick in progress), PSU
replacement deferred (see bom.md #23), server room + cable route. See `ROADMAP.md` Phase 0.

Storage drives are **on hand** as of 2026-10-21 (2x 4TB IronWolf + 970 Evo
Plus) — Phase 1 needs no drive purchases. See `HANDOFF.md` for current open
items before starting work.

**2026-10-25:** PSU voltages verified in spec at light load. The bagged modular
SATA cables are wired wrong for this PSU — **use factory-attached SATA and PCIe
cables only.** Drives, GPU and front fans are installed. Session log:
`docs/sessions/2026-10-25.md`.

## Hard constraints (do not design around these being flexible)

- **4× 3.5" drive bays** in the case — this is the storage ceiling, not SATA ports
- **32GB DDR4** (2x16, 2 slots free — 64GB possible but DDR4 pricing is painful) — ZFS ARC must be capped
- **8GB VRAM** on the server's primary GPU (RTX 2070 Super) — caps local models at ~7-8B at Q4 with modest context; 14B is off the table
- **750W PSU** — ~560W peak budget, little headroom for GPU upgrades
- **Modular PSU cables:** only the factory-attached SATA and PCIe cables are trusted.
  Never connect a drive or GPU to an untested modular cable
- **1GbE onboard** networking
- Server GPUs are mismatched generations and **will not pool** — treat as two
  independent workers, never as one large pool
- Server goes outside the living room on wired ethernet (ADR 0005) — room TBD; still an apartment, so noise and heat matter

## Decisions already made

See `docs/decisions/`. These are settled; do not re-litigate without new
information. Summary:

- Proxmox VE, not Unraid or bare Ubuntu (0001)
- ZFS mirrors, not raidz (0002)
- Orchestrator model lives on the *server*, not the desktop (0003)
- Recertified enterprise drives, not used consumer drives (0004)
- Server outside the living room, wired ethernet (0005)
- NVMe swapped between machines: 9100 Pro to desktop, 970 Evo Plus to server (0006)

## Conventions

- Hardware facts live in `docs/spec.md`. Purchase tracking in `bom.md`.
- Architecture in `docs/architecture.md`.
- New significant decisions get an ADR in `docs/decisions/` — numbered, dated.
- Anything unverified is marked `TBD` or `⚠️`. Do not silently fill these in
  with assumptions; flag them.

## Working style

- Push back on bad ideas rather than agreeing. Flag risks explicitly.
- One step at a time during troubleshooting. Do not stack multi-step
  instructions.
- Prices in this market move fast (2026 storage shortage). Verify current
  pricing rather than trusting figures in this repo.
