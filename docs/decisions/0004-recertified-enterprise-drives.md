# 0004 — Recertified enterprise drives over used consumer drives

**Status:** Accepted
**Date:** 2026-08-09

## Context

A local marketplace listing offered used consumer 3.5" drives: 1TB at $30, 2TB
at $60 — both ~$30/TB. Visible model numbers included WD1001FALS (1TB Caviar
Black, ~2009) and WD740 (74GB Raptor, mid-2000s).

## Decision

Buy recertified enterprise drives (12-16TB class, warrantied) instead.

## Rationale

- **Cost:** ~$30/TB versus roughly $12-18/TB for warrantied recerts — about
  double the per-TB cost for far worse hardware
- **Age:** 12-16 year old consumer drives. "Tested working" means it spun up
  once; it says nothing about remaining life
- **Architecture:** ~5TB now and growing would need 5+ drives at 1TB each,
  against 4 case bays and 6 SATA ports. Small drives burn the entire
  port/bay budget
- **Risk:** more drives multiplies annual failure probability, and some of this
  data is irreplaceable

## Consequences

- Higher per-drive cost, far lower drive count
- Warranty coverage on the drives holding irreplaceable data
- The only sane use for the cheap listing would be a single throwaway scratch
  disk, not primary storage
