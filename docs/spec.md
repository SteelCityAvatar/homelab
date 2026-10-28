# Hardware Spec Sheet

**Version:** 1.0
**Status:** POST verified 2026-10-05 — Phase 0 checks remaining (see ROADMAP)

---

## Node A — Desktop (existing, unchanged)

Doubles as the home theater PC. **The 4090 stays here.**

| Component | Spec |
|---|---|
| CPU | Intel i7-13700K |
| GPU | NVIDIA RTX 4090 (24GB) |
| RAM | 32GB DDR5-5600 |
| Storage | 8TB HDD — holds most media + personal files today, **single disk, no second copy** |
| SSD | Samsung 9100 Pro 1TB (PCIe 5.0) — took over from the 970 Evo Plus, which is now in the server (ADR 0006). Cloned from the 970; stable in daily use (confirmed 2026-10-27) |
| OS | Windows 11 |
| Display | Samsung S90D OLED |
| Audio | Onkyo TX-NR6050 AVR + 5.1 |

**Ecosystem role:** opportunistic escalation backend. Serves a 27–32B class
model over LAN when free; unavailable during gaming.

---

## Node B — Server

### On hand

| Component | Spec | Open |
|---|---|---|
| Motherboard | MSI MPG Z390 Gaming Plus (ATX, LGA1151) | POST verified 2026-10-05 |
| CPU | Intel i7-8700K (6C/12T, Coffee Lake) | — |
| RAM | 32GB DDR4 — 2x16GB 3200 kit (2 slots free) | XMP off, default SPD speed — ⚠️ likely 2133–2666, confirm in BIOS (8700K IMC is rated 2666; stability over speed for non-ECC ZFS). CL TBD. MemTest86 passed 2026-10-27. Spare 16GB kit on hand, deliberately not mixed in |
| PSU | Corsair TX750M (750W, semi-modular) | ~2013, approx. (owner recollection). ⚠️ ~13 yrs old — replacement deferred, conditions in bom.md #23. Light-load test 2026-10-25: 12V 11.97 / 5V 5.08 / 3.3V 3.35, all in spec. **Factory-attached SATA/PCIe cables only** — bagged modular SATA cables tested bad |
| GPU 1 | RTX 2070 Super 8GB (Turing, sm_75, ~215W) | Installed 2026-10-25 in the top x16 slot, factory-attached PCIe power; detected |
| GPU 2 | GTX 1060 6GB (Pascal, sm_61, ~120W) | — |
| Case | Cooler Master HAF XB EVO | 4x 3.5" per manufacturer spec. ⚠️ Not physically recounted |
| CPU cooler | Existing Cooler Master tower heatsink (~2019) + new 120mm fan | Fan is 4-pin PWM on `CPU_FAN`. Its 5V ARGB lead is left unplugged — never into the 12V `JRGB` header. Paste: Kryonaut — good contact confirmed, CPU 38°C during MemTest86 (2026-10-26) |
| Case fans | 2x be quiet! 140mm PWM (front intake) + stock 120mm (rear exhaust) | Front on `SYS_FAN` headers, PWM mode. Rear is old — ⚠️ plug type unknown; DC mode if 3-pin. 2x140 in vs 1x120 out = slight positive pressure |
| HDD x2 | Seagate IronWolf 4TB (CMR NAS) | Bought new 2026-10-21. On SATA1/SATA2, powered by the factory-attached SATA cable; detected 2026-10-25. Accidentally initialized as GPT in Windows — harmless, wiped at ZFS creation. ⚠️ Burn-in pending |
| NVMe | Samsung 970 Evo Plus **1TB** (PCIe 3.0) | Installed in M2_1. Holds boot + hot tier, so M2_2 stays free. Still carries the old desktop Windows install until Proxmox replaces it |

### Motherboard capability (verified)

| Feature | Spec |
|---|---|
| SATA III | 6 ports |
| M.2 | 2 slots |
| **M2_1 (NVMe)** | **Disables no SATA ports** — populate first |
| **M2_2 (NVMe or SATA)** | Disables SATA5 + SATA6 |
| PCIe | 1x x16, 1x x16 @ x4, 4x x1 |
| LAN | Realtek RTL8111H — 1GbE only |

### Case capability

| Feature | Spec |
|---|---|
| 3.5" bays | **4** (2 front hot-swap + 2 internal) |
| 2.5" mounts | 4 — converted from the 3.5" bays, **not additional** |
| 5.25" bays | 2 — expansion path via 3.5" cage adapter |
| Expansion slots | 7 |
| Max GPU length | 334mm |
| Max cooler height | 180mm (chassis mode) |
| Stock fans | 2x 120mm front; mesh front and top |

**Bay count:** 4x 3.5" per Cooler Master's spec page. ⚠️ Not physically recounted — confirm when installing drives.

---

## Target layout

```
M2_1  -> 970 Evo Plus 1TB: boot + hot tier       costs 0 SATA ports
M2_2  -> free for now                        (would cost SATA5, SATA6)

SATA1 -> IronWolf 4TB  \__ ZFS mirror = Vault tier
SATA2 -> IronWolf 4TB  /
SATA3 -> free  \__ 2 bays + 2 ports free for Bulk tier later
SATA4 -> free  /

PCIe x16 -> RTX 2070 Super
PCIe x4  -> GTX 1060
```

**Capacity gap:** the mirror gives 4TB usable against ~5TB of existing data.
Triage — irreplaceable files to the mirror first; media stays on the desktop's
9TB until bulk drives are bought. This also satisfies bom.md #23 condition (a),
which forbids deleting the desktop copy until a Vault backup exists off-server.

The 970 is 1TB, so the hot tier shares M2_1 with boot and M2_2 stays free (ADR 0006).

**Future expansion:** the 5.25" bays accept a 3.5" cage (-> ~7 drives), but that
exceeds 6 SATA ports and would require an HBA.

---

## Power budget (rough peak)

| Item | Watts |
|---|---|
| i7-8700K | ~130 |
| RTX 2070 Super | ~215 |
| GTX 1060 | ~120 |
| 4x HDD @ ~8W | ~32 |
| Board, fans, NVMe | ~60 |
| **Total** | **~560W** |

750W is adequate with headroom for transients. Limits future GPU upgrades.

---

## Constraints summary

| Constraint | Impact |
|---|---|
| 4x 3.5" bays | The real storage ceiling |
| 16 CPU PCIe lanes | Two GPUs at x16/x4 — fine for inference |
| 32GB DDR4 | Cap ZFS ARC; DDR4 pricing makes upgrades painful |
| Non-ECC RAM (8700K / Z390 can't do ECC) | RAM is the one unchecksummed stage of the data path: a bit flipped before ZFS checksums a write lands on both mirror disks with a valid checksum. Mitigations: MemTest86 before data goes in, regular snapshots, and treat checksum errors on **both** mirror disks at once as a RAM/controller signal, not a drive failure |
| 750W | ~560W peak; limits GPU upgrades |
| Mesh case, apartment | Heat needs somewhere to go — see Placement |
| 1GbE onboard | ~5TB migration is an overnight job |
| 2026 HDD shortage | ~46-50% price rise since Sept 2025; buy at need |

---

## Placement & noise

**Settled: the server goes outside the living room, on wired ethernet.**
See ADR 0005 for the reasoning. ⚠️ Room and cable route still TBD.

The HAF XB EVO is a high-airflow mesh case with no dampening, and ~560W peak is
mostly heat. Both mattered when living-room placement was on the table; with the
server elsewhere, noise mitigation largely drops away (ROADMAP Phase 4). Still
an apartment — check the chosen room is not a closed closet with no airflow.

---

## Contingency — if the board is dead

Any 300-series board runs the 8700K (Z370 / H370 / B360 / B365 / Z390). No need
to pay the Z premium for a server.

**Requirements:** 6x SATA, and re-check that board's M.2/SATA sharing table — it
is board-specific and drives the target layout above.
