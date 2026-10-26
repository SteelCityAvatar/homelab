# Bill of Materials

**Prices intentionally omitted.** The 2026 storage/memory shortage moves fast —
look up current pricing rather than trusting a number written here.

Item numbers are stable IDs referenced elsewhere in the repo. Don't renumber.

Last updated 2026-10-26.

---

## Have — server

| # | Part | Spec | Status | Notes |
|---|---|---|---|---|
| 1 | Motherboard | MSI MPG Z390 Gaming Plus | Installed | POST verified 2026-10-05 |
| 2 | CPU | i7-8700K | Installed | — |
| 3 | RAM | 32GB DDR4 — 2x16GB 3200 kit | Installed | XMP off, default SPD speed (⚠️ confirm in BIOS). **MemTest86 pending.** Spare 16GB kit on hand, deliberately not mixed in |
| 4 | PSU | Corsair TX750M (750W, semi-modular) | Installed | ~2013, ~13 yrs old. Light-load voltages in spec (2026-10-25). Replacement deferred — conditions in #23 |
| 5 | GPU | RTX 2070 Super 8GB | Installed | Top x16 slot, **factory-attached** PCIe power. Detected 2026-10-25 |
| 6 | GPU | GTX 1060 6GB | On hand | Not installed yet |
| 7 | Case | Cooler Master HAF XB EVO | In use | 4x 3.5" per manufacturer spec. ⚠️ Not physically recounted |
| 8 | CPU cooler | Existing Cooler Master tower heatsink (~2019) | Installed | With the fan from #12 |
| 12 | CPU fan | 120mm, 4-pin PWM | Installed 2026-10-24 | On `CPU_FAN`. Its 5V ARGB lead is left unplugged — never into the 12V `JRGB` header |
| 13 | Front case fans | 2x be quiet! 140mm PWM | Installed 2026-10-25 | Intake, on `SYS_FAN` headers, PWM mode |
| — | Rear case fan | Stock 120mm | In use | Old. ⚠️ Plug type unknown — DC mode if 3-pin. Replace if it rattles (#30) |
| 15 | Thermal paste | Thermal Grizzly Kryonaut | Arrived 2026-10-22 | ⚠️ Applied — not explicitly confirmed |
| 16 | SATA data cables | 4 | 2 in use | SATA1 + SATA2 |
| 17 | SATA power | **Factory-attached** SATA cable on the TX750M | In use | Verified with a multimeter 2026-10-25. Powers both IronWolfs |
| 18 | USB stick | 8GB+ | On hand | MemTest86 now, Proxmox installer later |
| 24 | HDD x2 | Seagate IronWolf 4TB (CMR NAS) | Installed | Bought new 2026-10-21. SATA1/SATA2, detected. Vault mirror. ⚠️ Burn-in pending — inside the return window. Keep vs. return still open |
| 25 | NVMe | Samsung 970 Evo Plus 1TB | Installed | M2_1. Boot + hot tier, so M2_2 stays free. Still holds the old desktop Windows install. ⚠️ SMART/TBW check pending |

## Have — desktop

| # | Part | Spec | Status | Notes |
|---|---|---|---|---|
| 26 | NVMe | Samsung 9100 Pro 1TB (PCIe 5.0) | Installed | Bought new 2026-10-21. Replaced the 970 (ADR 0006). ⚠️ Stable boot not explicitly confirmed |

## Tools

| # | Part | Spec | Notes |
|---|---|---|---|
| 27 | Multimeter | Triplett 2030-C, autoranging, built-in leads | Bought 2026-10-25. See `docs/runbooks/verify-modular-psu-cables.md` |

## Do not use ⚠️

| # | Part | Why |
|---|---|---|
| 28 | Bagged modular SATA cables (2 tested) | **BAD for the TX750M** (2026-10-25): 3.3V in the 5V position, 12V where 3.3V belongs. Would kill a drive. Label them and keep them out of the build |
| 29 | Bagged modular PCIe cables | **Untested.** Use only the factory-attached PCIe cables. Test any modular cable before it goes near a GPU |

The modular CPU 8-pin from the same bag **does** work — the board POSTs and
boots on it — so that bag is mixed, not uniformly wrong.

---

## Need

| # | Part | Qty | Priority | Notes |
|---|---|---|---|---|
| 19 | Ethernet cable | 1 | **Phase 1 — now** | The server has no network. Proxmox is managed through a web UI over the LAN, so this blocks setup. Length depends on the room (ADR 0005) |
| 22 | UPS | 1 | Before the Vault holds data | Protects ZFS + irreplaceable data; triggers clean shutdown |
| 23 | PSU replacement | 1 | Deferred, still recommended | See below |
| 11 | HDD, recertified enterprise | 2 | Later | Bulk tier — 2 bays + 2 SATA ports free. Buy when prices allow |
| 14 | Anti-vibration HDD grommets/screws | 1 set | Optional | Meaningful in a mesh case |
| 20 | 2.5GbE NIC | 2 | Optional, Phase 4 | One per machine. Check x1 slot access with the GPUs installed |
| 30 | Rear case fan | 1 | Optional | 120mm be quiet! PWM, if the stock exhaust rattles |

**#23 — PSU replacement.** Deferred 2026-10-05, still recommended. The TX750M is
~13 years old. Conditions while deferred:

- **(a)** Don't delete the desktop copy of any irreplaceable data until a Vault
  backup exists **off** the server.
- **(b)** Replace immediately on random reboots or shutdowns under GPU load, coil
  whine, or failure to power on. ⚠️ A PSU noise was heard 2026-10-25 but not
  characterized — check it against this.
- **(c)** Re-test under sustained GPU load before Phase 2.

Light-load test 2026-10-25: **12V 11.97 / 5V 5.08 / 3.3V 3.35**, all within ±5%.
Buy a quality 750–850W unit. Its modular cables won't be compatible with the
TX750M's, so use only the new unit's own cables.

## Resolved — not needed

| # | Part | Why |
|---|---|---|
| 9 | NVMe, boot | The 970 (#25) covers boot. ADR 0006 |
| 10 | NVMe, models/hot | The 970 is 1TB, so the hot tier shares M2_1 (2026-10-24) |
| 21 | 2.5GbE switch | Direct point-to-point link between the nodes instead |

---

## Phasing

**Phase 1 needs one purchase: an Ethernet cable (#19).** Everything else for it
is on hand. Get the UPS (#22) before the Vault holds data that exists nowhere
else. Bulk HDDs (#11) can wait for better prices.
