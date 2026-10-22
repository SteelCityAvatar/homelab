# Runbook: Bench-test the motherboard for POST

**Purpose:** Determine whether the MSI MPG Z390 Gaming Plus is functional before
any money is spent. This blocks everything else.

**Time:** ~15 minutes.

## Why bench, not in-case

Isolates faults. A short against the case standoffs looks identical to a dead
board, and you do not want to diagnose that with 4 drives and 2 GPUs installed.

## What you need

- Motherboard, i7-8700K, CPU cooler, one stick of DDR4
- PSU, monitor + HDMI cable, screwdriver
- The board's cardboard box (an anti-static work surface)

## Advantages on this board

- The **8700K has integrated graphics** and the board has HDMI/DVI out — no GPU
  needed for this test
- The board has **EZ Debug LEDs**: four labeled lights (CPU, DRAM, VGA, BOOT).
  Whichever stays lit is the stage that failed. That turns "it doesn't work"
  into an actual diagnosis.

## Procedure

1. Place the board on its cardboard box. **Not** on carpet, not on metal.
2. Mount the CPU and cooler.
3. Install **one** stick of RAM, in the slot nearest the CPU (DIMMA2 on most MSI
   boards — check the silkscreen).
4. Connect the 24-pin ATX and the 8-pin CPU power.
5. Monitor into the board's HDMI port.
6. No GPU. No drives. No front-panel connections beyond what step 7 needs.
7. Briefly short the two power-switch pins on the front-panel header with a
   screwdriver.

## Interpreting the result

| Outcome | Meaning | Next |
|---|---|---|
| BIOS screen | Board is alive | Proceed to Phase 1 |
| CPU LED lit | CPU or CPU power fault | Reseat CPU, check 8-pin |
| DRAM LED lit | RAM fault | Try the other stick / another slot |
| VGA LED lit | Graphics init fault | Unexpected with iGPU — note it |
| BOOT LED lit | Expected — no boot device attached | Board is fine |
| Nothing at all | No power or dead board | Check PSU first, then assume board |

**Note:** a lit BOOT LED with a BIOS screen is normal here. There is no boot
device attached, by design.

## Also record while you have it open

- RAM configuration: 2x16 or 4x8? (determines upgrade path)
- PSU model and approximate age (see spec.md — this is a real risk item)
