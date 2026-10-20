---
title: Power Budget
---

# Power Budget

**RAS 304 – Individual Datasheet (Kaushik) – Driver Nano 3: Linear Actuator B + IMU**

> Draft values for my individual board only. Part 2 is optional for the initial submission but required for the external design review. Values marked "est." should be replaced with measured values once the board is built. These numbers still need to be copied into the course's power budget spreadsheet template.

## Power architecture

- One external source: my own 9 V wall adapter (1 A) on the barrel jack, through a 1 A fuse.
- Two rails: **9 V** (raw adapter voltage, feeding only the regulator) and **5 V** (L7805CV linear regulator).
- The 5 V rail powers the DRV8833 motor supply (VM), the N20 gearmotor, and the Curiosity Nano (VBUS pin).
- The Curiosity Nano makes its own 3.3 V (VTG), which powers the LIS3DH and the slide-potentiometer reference. That load is counted on the 5 V rail because the Nano's regulator draws it from the 5 V input.

## Section A. Components that consume power (absolute maximum current)

Regulators, the power source, switches, the fuse, the LED and passives are not listed.

| Component | Supply voltage | Max current (mA) | Basis |
|---|---|---|---|
| N20 6 V gearmotor with M3 lead screw (Actuator B) | 5 V via DRV8833 | 600 | Stall current est.: the seller does not publish it. A similar 6 V N20 lead-screw motor (DFRobot FIT0577) lists 420 mA stall at 6 V; 600 mA is a conservative allowance for the faster 300 RPM motor. Measure on the bench |
| PIC18F57Q43 Curiosity Nano (MCU, on-board debugger, 3.3 V regulator) | 5 V | 100 | Conservative allowance (est.); not the board's own 500 mA regulator limit |
| DRV8833PWPR (own supply current, motor current not included) | 5 V (VM) | 5 | Allowance (est.); the datasheet gives only a 1.6 µA sleep current in the portion I read |
| LIS3DHTR accelerometer | 3.3 V (from Nano) | 0.5 | Allowance (est.) for operating mode |
| Bourns PTA2043 slide pot, 10 kΩ ±20% | 3.3 V (from Nano) | 0.5 | 3.3 V ÷ 8 kΩ (worst-case low resistance) is about 0.41 mA, rounded up |

Note: the DRV8833 does not itself draw the motor current; the motor does, so the driver is not counted twice. The power LED is not listed because the assignment excludes LEDs.

## Section B. Rail assignment with 25% safety margin

| Rail | Component | Current (mA) |
|---|---|---|
| **5 V** | N20 gearmotor (stall) | 600 |
| 5 V | Curiosity Nano | 100 |
| 5 V | DRV8833 supply | 5 |
| 5 V | LIS3DHTR (via Nano 3.3 V) | 0.5 |
| 5 V | Slide pot (via Nano 3.3 V) | 0.5 |
| | **5 V rail total** | **706.0** |
| | **× 1.25 safety margin** | **882.5** |
| **9 V** | No direct loads | 0 |

## Section C. Regulator or source choice per rail

| Rail | Choice | Rationale |
|---|---|---|
| 5 V | ST L7805CV (TO-220) with heatsink | Specified for 7 to 18 V input, so the 9 V adapter has 4 V of headroom over its 2 V dropout. Rated up to 1.5 A against a 0.88 A load with margin. Has internal current limiting and thermal shutdown. Compared with the LM2940CT-5.0 in the [Component Selection](../02-Component-Selection/Component-Selection.md) page. |
| 9 V | Straight from the wall adapter | Only feeds the regulator, so no second regulator is needed |

Regulator thermal check:

| Quantity | Value |
|---|---|
| Load current including margin | 882.5 mA |
| Regulator quiescent current (datasheet, about 8 mA) × 1.25 | 10 mA |
| Power dissipated: (9 V − 5 V) × 0.8825 A | 3.53 W |
| Junction rise at 50 °C/W (TO-220, no heatsink) | about 177 °C, so it would hit thermal shutdown |
| Junction-to-ambient resistance needed to stay under 125 °C at 40 °C ambient | 24 °C/W or lower |

The regulator therefore **needs a heatsink**. Junction-to-case is about 5 °C/W, so a heatsink of roughly 15 °C/W or better is required. At the nominal 0.6 A motor load without margin the dissipation is still about 2.8 W, so the heatsink is needed in normal operation too. My Block Diagram page says a "small" heatsink; it should be sized to this calculation.

## Section D. External power source

| Item | Value |
|---|---|
| Source choice | My existing 9 V 1 A regulated wall adapter |
| Current into the L7805CV input: (706 + 8) × 1.25 | 892.5 mA |
| **Total required from adapter** | **892.5 mA** |
| Adapter capacity | 1000 mA |
| **Total remaining current available** | **107.5 mA (not negative)** |

This check depends on the motor current staying near 600 mA. The DRV8833 current limit in my Block Diagram page is about 1 A (a 0.2 Ω sense resistor). A stalled motor could then draw up to 1 A, giving (1000 + 114) × 1.25 = 1,393 mA from the adapter, which is more than it can supply. The limit must be lowered: a sense resistor of about 0.3 Ω gives a limit near 0.67 A, which gives 976 mA from the adapter and keeps the remaining current positive (about 24 mA).

## Section E. Battery life

Not applicable. The design runs from a wall adapter and has no battery.

## Open items before the design review

1. Measure the stall current of the 300 RPM N20 motor at 5 V and replace the 600 mA estimate.
2. Change the DRV8833 sense resistor from 0.2 Ω to about 0.3 Ω so the current limit is near 0.67 A, and update the Block Diagram page.
3. Choose and price a TO-220 heatsink of about 15 °C/W or better, or consider powering the motor driver from the 9 V rail with the PWM duty capped near 66% (a 6 V motor on 9 V), which would remove most of the regulator's heat.
4. Replace the Nano (100 mA), DRV8833 (5 mA) and LIS3DH (0.5 mA) estimates with measured current.
5. Copy these numbers into the course power budget spreadsheet template.
