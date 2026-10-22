# 0006 — Swap NVMe drives between the two machines

**Status:** Accepted
**Date:** 2026-10-21

## Context

Drives confirmed on hand 2026-10-21:

- Samsung 9100 Pro 1TB — PCIe 5.0 x4, rated ~14,700 MB/s read. Not installed
- Samsung 970 Evo Plus — PCIe 3.0 x4, the desktop's only SSD (capacity TBD)

This resolves the open option in `bom.md` #10 ("reuse the desktop's 1TB Samsung
— model TBD"). The desktop's drive is the 970 Evo Plus; the 9100 Pro is the
loose one.

## Decision

The 9100 Pro goes in the desktop (Node A). The 970 Evo Plus moves to the server
(Node B).

## Rationale

**Interface mismatch.** The Z390's M.2 slots are PCIe 3.0, ceiling ~3,500 MB/s.
A PCIe 5.0 drive there runs at roughly a quarter of its rated speed.

**The server benefits least from fast NVMe.** Model weights load once into VRAM
and the drive then idles. Drive speed affects load time, not inference. Putting
the premium drive in the server wastes it twice — once on the interface, once on
the workload.

**The desktop is thin on fast storage.** 9TB plus a single 970 Evo Plus, while
running games *and* serving a 27-32B escalation model.

**The 970 is a native match for PCIe 3.0.** Nothing is wasted in that slot.

## Consequences

- The 970 is the desktop's only SSD and presumably its Windows boot drive, so
  this is a clone-or-reinstall job, with the 9100 Pro becoming the new boot
  drive. One-time cost.
- ⚠️ Check whether the desktop board has a PCIe 5.0 M.2 slot. If Gen 4 only, the
  9100 Pro still delivers ~7,000 MB/s — double what Z390 allows.
- ⚠️ 970 Evo Plus capacity unknown. 250-500GB = boot only, and the hot tier needs
  its own drive in M2_2 (costing SATA5/6). 1-2TB = covers boot and hot together.
- SMART health / TBW check on the 970 before trusting it, per bom.md #10.
