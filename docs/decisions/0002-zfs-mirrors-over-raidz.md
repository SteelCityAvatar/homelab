# 0002 — ZFS mirrors over raidz

**Status:** Accepted
**Date:** 2026-08-09

## Context

Storage is being bought during the 2026 HDD shortage (~46-50% price rise since
September 2025, hyperscaler demand consuming supply). Capacity will be added
incrementally as budget allows.

## Decision

Mirrored vdevs, not raidz.

## Rationale

Counterintuitive during a shortage, since mirrors cost 50% of raw capacity. But
the **growth path** dominates:

- Mirrors expand a pair at a time — buy drives as prices allow
- raidz vdev width is effectively locked at creation; expansion exists now but
  carries caveats
- Mirrors resilver dramatically faster, which matters when a replacement drive
  is a long-lead-time item

Capacity efficiency is the wrong thing to optimize when drive availability is
the binding constraint.

## Consequences

- Vault tier is a 2-drive mirror; Bulk tier is unmirrored (re-acquirable data)
- Adding capacity means buying drives in pairs
