# Bill of Materials

**Prices intentionally omitted.** The 2026 storage/memory shortage moves fast —
look up current pricing rather than trusting a number written here.

## Have

| # | Part | Spec | Open question |
|---|---|---|---|
| 1 | Motherboard | MSI MPG Z390 Gaming Plus | POST verified 2026-10-05 |
| 2 | CPU | i7-8700K | — |
| 3 | RAM | 32GB DDR4 — 2x16GB 3200 kit | XMP off, default SPD speed (⚠️ confirm in BIOS). CL TBD. Memtest pending. Spare 16GB kit on hand, not used |
| 4 | PSU | Corsair TX750M | ~2013 (approx.). ⚠️ ~13 yrs — past the 8-yr threshold below. Bench/testing use only; see #23 |
| 5 | GPU | RTX 2080 Ti 11GB | — |
| 6 | GPU | GTX 1060 6GB | — |
| 7 | Case | HAF XB EVO | 4x 3.5" per manufacturer spec. ⚠️ Not physically recounted |
| 8 | CPU cooler | Existing aftermarket tower heatsink (name brand, ~2019) | Model unknown. Fits a 120mm fan. Fan missing — add one (#12) |
| 24 | HDD x2 | Seagate IronWolf 4TB (CMR NAS) | Bought new 2026-10-21. ⚠️ Burn-in pending (ROADMAP Phase 1). Becomes the Vault mirror |
| 25 | NVMe | Samsung 970 Evo Plus | ⚠️ Capacity TBD. Moving in from the desktop — ADR 0006. SMART/TBW check first |
| 26 | NVMe | Samsung 9100 Pro 1TB | Bought new 2026-10-21. PCIe 5.0 — goes to the **desktop**, not the server (ADR 0006) |

## Need — storage

| # | Part | Qty | Notes |
|---|---|---|---|
| 9 | ~~NVMe, boot~~ | 0 | **Resolved** — the 970 Evo Plus (#25) covers boot. ADR 0006 |
| 10 | NVMe, models/hot | 0 or 1 | **Only if the 970 is 250-500GB.** Goes in M2_2, costs SATA5/6. The open "1TB Samsung" option is resolved: that drive is the 970, and the 9100 Pro goes to the desktop instead (ADR 0006) |
| 11 | HDD, recertified enterprise | 0 now, 2 later | **Vault tier is covered by the IronWolf pair (#24).** These become the Bulk tier — 2 bays + 2 SATA ports free. Buy when prices allow |

## Need — cooling & noise

| # | Part | Qty | Notes |
|---|---|---|---|
| 12 | 120mm PWM fan for existing heatsink | 1 | **Plan (2026-10-05):** revive the existing heatsink rather than buy a new cooler — server is off the living room, so noise matters less. Fallback if thermals disappoint: Thermalright Peerless Assassin / Phantom Spirit (~155mm, max 180mm) |
| 13 | Quiet 120mm case fans | 2-3 | Low RPM |
| 14 | Anti-vibration HDD grommets/screws | 1 set | Meaningful in a mesh case |
| 15 | Thermal paste | 1 | Thermal Grizzly Kryonaut — **bought, arriving 2026-10-22** |

## Need — cables & small parts

| # | Part | Qty | Notes |
|---|---|---|---|
| 16 | SATA data cables | 4 | |
| 17 | SATA power splitters | TBD | Count the PSU's SATA connectors first |
| 18 | USB stick 8GB+ | 1 | Proxmox installer |

## Need — network

| # | Part | Qty | Notes |
|---|---|---|---|
| 19 | Ethernet cable | 1 | Length depends on room — TBD (ADR 0005) |
| 20 | 2.5GbE NIC | 2 | Optional — one per machine. Check x1 slot access with 2 GPUs installed |
| 21 | 2.5GbE switch | 0 | **Skip.** Run a direct point-to-point link between server and desktop on its own subnet |

## Need — protection

| # | Part | Qty | Notes |
|---|---|---|---|
| 22 | UPS | 1 | Protects ZFS + irreplaceable data; triggers clean shutdown |
| 23 | PSU replacement | 1 | **Deferred (2026-10-05), still recommended.** Existing TX750M is ~13 yrs old. Conditions while deferred: (a) do not delete the desktop copy of any irreplaceable data until a Vault backup exists **off** the server; (b) replace immediately on random reboots/shutdowns under GPU load, coil whine, or failure to power on; (c) re-evaluate before Phase 2 puts both GPUs under sustained load. Quality 750-850W unit. Modular cables are not cross-compatible — use only the new unit's cables |

---

## Pre-purchase checks

**PSU age and model.** The one real risk item. Electrolytic capacitors degrade
with heat and years, and the PSU is the single component whose failure can take
the board, both GPUs, and the drives with it. If the 750W unit is 8+ years old
and has run hard, replace it before trusting it with irreplaceable data.

**RAM configuration.** 4x8 means every slot is full and any increase requires
replacing all of it — painful at current DDR4 prices. 2x16 leaves a clean path
to 64GB. RAM is the real ceiling on this build.

---

## Phasing

Do not buy everything at once. With the IronWolf pair and the 970 on hand,
**Phase 1 needs only #12 (120mm fan) and #16-18 (cables, USB stick)** — paste (#15) is bought —
plus #10 only if the 970 turns out to be 250-500GB. Bulk HDDs (#11) can wait for
better prices. Everything else follows once the machine is running.
