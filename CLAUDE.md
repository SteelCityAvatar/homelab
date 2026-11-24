# CLAUDE.md

Context for Claude Code working in this repo.

## What this project is

Build and operate a home server ("Node B") that acts as the always-on half of a
two-machine ecosystem. The other half ("Node A") is an existing Windows 11
desktop that doubles as a home theater PC and must remain untouched.

The server's job: bulk storage, local model serving, and home automation — so
that Node A can stay a gaming/theater machine.

## Current status

**Phase 1 (storage) and Phase 2 (model serving) in progress.** Proxmox VE 9.2.24 on
the 970, web UI `https://192.168.1.169:8006`, SSH `root@192.168.1.169` (key auth from
the desktop: Claude Code can run server commands as
`ssh -o BatchMode=yes root@192.168.1.169 "<cmd>"`, each one approved). Tailscale on
the host for remote access (ADR 0008). The NVIDIA driver is installed (595.104.02,
verified) and the model-serving container (ID 200, `192.168.1.164`) exists; GPU
passthrough is next. Data drives: the 2x 4TB IronWolfs are kept; 2x 14TB recerts are
a later purchase, not ordered (plan in `HANDOFF.md`).
**Read `HANDOFF.md` first** for the current step and open items. To bring the repo
up to date after work, use the `update-repo` skill.

## Hard constraints (do not design around these being flexible)

- **4× 3.5" drive bays** in the current case — the storage ceiling *for now*. This
  chassis is effectively a test bench; more drive space is planned later. Keep the
  ZFS pool portable (it moves to new hardware with `zpool export` / `import`)
- **32GB DDR4** (2x16, 2 slots free — 64GB possible but DDR4 pricing is painful) — ZFS ARC must be capped
- **8GB VRAM** on the server's primary GPU (RTX 2070 Super) — caps local models at ~7-8B at Q4 with modest context; 14B is off the table
- **750W PSU**, ~13 years old — ~440W peak budget; load-test before sustained GPU work
- **Modular PSU cables:** only the factory-attached SATA and PCIe cables are trusted.
  Never connect a drive or GPU to an untested modular cable
- **1GbE onboard** networking
- **One server GPU** (RTX 2070 Super). The GTX 1060 is out (ADR 0007)
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
- NVIDIA driver from NVIDIA's `.run` installer with DKMS; 2070 Super only (0007)
- Remote access through Tailscale; no inbound port forwards (0008)

## Conventions

- Hardware facts live in `docs/spec.md`. Purchase tracking in `bom.md`.
- **`README.md` is the public front page.** Update its status table, roadmap row
  and build-log highlights whenever a phase moves or a notable incident is resolved.
- Architecture in `docs/architecture.md`.
- New significant decisions get an ADR in `docs/decisions/` — numbered, dated.
- **Log every command.** Each PowerShell (desktop) and bash (server) command goes in
  the current `docs/sessions/<date>.md` under "Command log": exact command,
  read-only vs. destructive, outcome. Include failed/interrupted ones. Append as you go.
- **The repo is public. Write it as an engineer's build log:** impersonal voice, no
  "the owner", no references to AI chats or sessions, no personal or household
  details (people, retailers, receipts). Record operator slips only when there's a
  reusable lesson, and state the lesson, not the slip. Real incidents get honest
  write-ups.
- `docs/notes/` holds reference explanations (the *why* behind concepts: apt,
  ZFS and RAM, PSI graphs, networking, PSU cables). **Gitignored — local only.**
  When something is explained in a session and worth keeping, add or extend a
  note there rather than in the session log. **Every note starts with an
  `## Acronyms` list** spelling out each acronym it uses; keep it (and the full list
  in `docs/notes/README.md`) in sync when a note gains new terms. The notes also
  have a phone copy; after changing them, rebuild and republish it (steps in
  `docs/notes/README.md`).
- Anything unverified is marked `TBD` or `⚠️`. Do not silently fill these in
  with assumptions; flag them.

## Working style

- Push back on bad ideas rather than agreeing. Flag risks explicitly.
- One step at a time during troubleshooting. Do not stack multi-step
  instructions.
- Prices in this market move fast (2026 storage shortage). Verify current
  pricing rather than trusting figures in this repo.
