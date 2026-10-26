# Runbook: Verify a modular PSU cable with a multimeter

**Purpose:** Confirm a detachable PSU cable is wired for *this* PSU before it
touches a drive or GPU. Modular connectors from different PSUs often fit the same
port while carrying different voltages on each pin. Fitting proves nothing.

**History:** On 2026-10-25 this test caught two bagged modular SATA cables wired
wrong for the TX750M — 12V where 3.3V belongs. They would have killed a new
IronWolf on first power-up.

**Time:** ~10 minutes per cable.

## The reference rule

**A factory-attached cable of the same type is the truth.** It can't be mismatched.
Measure the factory plug first, then the modular one at the same positions. Any
difference means the modular cable is wrong.

## Tools

- Multimeter (Triplett 2030-C: autoranging, built-in leads)
- Dial on **DC volts** — the V with a straight line, *not* the wavy one
- **Don't press RANGE.** If you do, turn the dial OFF and back on to restore AUTO

## Setup

1. PSU rocker **off**. No drives or GPU connected to the cable under test.
2. Board connected (24-pin + CPU 8-pin) so the PSU can be switched on with a real load.
3. Plug the cable under test into the PSU. Let its far end hang free, touching no metal.
4. Black probe on bare metal: a PSU mounting screw, or the metal shell of a rear USB port.
5. Rocker on, then tap the `JFP1` power-switch pins.

## SATA power plug

Hold the plug with the slot facing you and the **L-shaped hook on the left**.
Fifteen contacts in five groups of three, one group per wire:

| Position | Rail | Correct reading |
|---|---|---|
| Hook end (contacts 1–3) | 3.3V | 3.14–3.47V, or 0V if the cable has no 3.3V wire |
| Contacts 4–6 | Ground | ~0V (a few mV either side is fine) |
| **Center (contacts 7–9)** | **5V** | **4.75–5.25V** |
| Contacts 10–12 | Ground | ~0V |
| Far end (contacts 13–15) | 12V | 11.40–12.60V |

Orientation verified on the TX750M's factory-attached SATA cable, 2026-10-25.

**Tips:**

- Counting tiny contacts is error-prone. The **two ends and the dead center** are
  easy to find without counting, and they're the readings that matter most.
- Wait for the reading to settle. A jumping number means the probe isn't seated.
- Bridging two contacts may trip the PSU off. That's its protection — harmless.
  Rocker off, wait 30 seconds, retry.

## PCIe power plug (6-pin / 8-pin)

Every pin reads either ~12V or ~0V. Measure a **factory-attached PCIe plug** pin
by pin, then the modular one in the same orientation. The 12V pins must be in the
same positions on both.

## Verdict

- **Pass:** every position matches the table or the factory reference, within spec.
- **Fail:** any voltage on a ground position, 5V anywhere but the SATA center, or
  any mismatch with the factory plug. Label the cable **BAD — do not use** and keep
  it away from the build.

Rocker off between cables.
