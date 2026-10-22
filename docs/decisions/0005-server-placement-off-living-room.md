# 0005 — Place the server outside the living room, wired ethernet

**Status:** Accepted
**Date:** 2026-10-05

## Context

The HAF XB EVO is a high-airflow mesh case with no sound dampening. Peak system
draw is ~600W, nearly all of which ends up as heat in the room. The living room
is also where Node A serves as the home theater PC, so server noise competes
directly with movie audio. `docs/spec.md` listed three options; a different room
with an ethernet run was the stated preference.

## Decision

Put Node B in a room other than the living room, connected by wired ethernet.
Wi-Fi is not an option for the server.

## Rationale

- **Noise:** HDD seek chatter and a 2080 Ti under load are audible through
  mesh. Mitigation (grommets, spindown, undervolting) reduces but does not
  remove that noise
- **Heat:** ~600W peak into a living room is a real problem in an apartment
  in summer
- **Wired only:** a ~5TB migration and ongoing LAN model serving are not
  workable over Wi-Fi

## Open

- ⚠️ Which room — TBD
- ⚠️ Cable route — TBD. Apartment: running cable through walls may need
  landlord permission. Alternatives are flat cable along the baseboards or
  under a door, or MoCA if there is coax in the walls. A switch does not
  remove the need for a cable run; it only adds ports at the far end
- ⚠️ This affects the Phase 4 2.5GbE direct link (`docs/architecture.md`,
  `bom.md` #20-21): with the nodes in different rooms, that link needs its own
  long cable run, or a 2.5GbE switch instead. Revisit then

## Consequences

- Living-room noise mitigation (`ROADMAP.md` Phase 4) mostly drops away
- Ethernet cable length (`bom.md` #19) depends on the chosen room
- Heat moves to the chosen room — check it isn't a closed closet with no airflow
