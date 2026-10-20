---
title: Power Budget
---

# Power Budget

**RAS 304 – Individual Datasheet (Kaushik) – Driver Nano 2: Linear Actuator A + Pressure Sensor**

> Draft values for the individual board only. Part 2 is optional for the initial submission but required for the external design review. Estimates marked "est." should be replaced with measured values once the board is built. These numbers still need to be copied into the course's power budget spreadsheet template.

## Power architecture

- One external source: a 9 V wall adapter on the barrel jack (the course supply).
- Two rails: **9 V** (raw adapter voltage) and **5 V** (L7805CV linear regulator from the 9 V rail).
- The Curiosity Nano takes the 5 V rail on its VBUS pin and makes its own 3.3 V (VTG), which powers the MCP6002 and the actuator potentiometer reference, so all analog signals stay within the ADC range. That 3.3 V load is counted on the 5 V rail below because the Nano's regulator draws it from the 5 V input.

## Section A. Components that consume power (absolute maximum current)

Regulators, the power source, switches and passives are not listed here.

| Component | Supply voltage | Max current (mA) | Basis |
|---|---|---|---|
| PIC18F57Q43 Curiosity Nano (MCU, on-board debugger, 3.3 V regulator) | 5 V | 100 | Conservative allowance (est.); the board's own 3.3 V regulator is rated for a 500 mA load, so this is not its limit. Measure on the bench |
| SN754410NE, logic supply (VCC1) | 5 V | 70 | Highest quiescent figure in the datasheet table (outputs low); confirm units against the datasheet |
| SN754410NE, motor supply quiescent (VCC2) | 9 V | 33 | Highest quiescent figure in the datasheet table; the motor current itself is listed on the actuator line below |
| MCP6002-I/P dual op-amp | 3.3 V (from Nano) | 0.5 | Allowance for two amplifiers (est.); typical supply current is on the order of 0.1 mA per amplifier |
| FSR 402 divider / bias network | 3.3 V (from Nano) | 1 | Design allowance for the divider resistor (est.) |
| PQ12-P potentiometer reference (5 kΩ ±50%) | 3.3 V (from Nano) | 2 | 3.3 V divided by the 2.5 kΩ low-tolerance case is about 1.3 mA, rounded up |
| Linear Actuator A (Actuonix PQ12-P 6 V) | 9 V via driver | 600 | Datasheet stall current is 550 mA at 6 V; with about 6.4 V at the motor in the worst case this is about 590 mA, rounded to 600 |

Note: the SN754410 does not itself draw the motor current, the actuator does, so the driver is not counted twice. The power LED is not listed because the assignment excludes LEDs.

## Section B. Rail assignment with 25% safety margin

| Rail | Component | Current (mA) |
|---|---|---|
| **5 V** | Curiosity Nano | 100 |
| 5 V | SN754410 VCC1 | 70 |
| 5 V | MCP6002 (via Nano 3.3 V) | 0.5 |
| 5 V | FSR bias (via Nano 3.3 V) | 1 |
| 5 V | Actuator potentiometer reference (via Nano 3.3 V) | 2 |
| | **5 V rail total** | **173.5** |
| | **× 1.25 safety margin** | **216.9** |
| **9 V** | SN754410 VCC2 quiescent | 33 |
| 9 V | Linear Actuator A (stall) | 600 |
| | **9 V rail direct loads total** | **633** |
| | **× 1.25 safety margin** | **791.3** |

## Section C. Regulator or source choice per rail

| Rail | Choice | Rationale |
|---|---|---|
| 5 V | ST L7805CV (TO-220) | Specified for 7 to 18 V input, so the 9 V adapter has 4 V of headroom over its 2 V dropout. Rated up to 1.5 A against a 0.22 A load with margin. Has internal current limiting and thermal shutdown. Compared with the LM2940CT-5.0 in the [Component Selection](../02-Component-Selection/Component-Selection.md) page. |
| 9 V | Straight from the wall adapter | The actuator and driver run directly from the adapter, so no second regulator is needed |

Regulator check:

| Quantity | Value |
|---|---|
| Load current including margin | 216.9 mA |
| Regulator quiescent current (datasheet, about 8 mA) × 1.25 | 10 mA |
| Power dissipated: (9 V − 5 V) × 0.217 A | 0.87 W |
| Junction rise at 50 °C/W (TO-220, no heatsink) | about 43 °C |
| Junction temperature at 40 °C ambient | about 83 °C (limit 125 °C) |

## Section D. External power source

| Item | Value |
|---|---|
| Source choice | SparkFun 9 V 2 A regulated wall adapter |
| Current into 9 V rail loads (with margin) | 791.3 mA |
| Current into the L7805CV input: (173.5 + 8) × 1.25 | 226.9 mA |
| **Total required from adapter** | **1018.1 mA** |
| Adapter capacity | 2000 mA |
| **Total remaining current available** | **981.9 mA (not negative)** |

For comparison, a 1 A adapter would leave 1000 − 1018.1 = **−18.1 mA**, which fails the check. That is why the 2 A adapter was selected. The adapter's plug polarity must be confirmed before first power-up (see the [Component Selection](../02-Component-Selection/Component-Selection.md) page).

## Section E. Battery life

Not applicable. The design runs from a wall adapter and has no battery.

## Open items before the design review

1. Replace the Nano (100 mA) and op-amp (0.5 mA) estimates with measured current.
2. Confirm the SN754410 quiescent currents and units against the datasheet tables.
3. Measure the motor-terminal voltage and stall current of the 6 V actuator on the 9 V rail, and cap the PWM duty accordingly.
4. Copy these numbers into the course power budget spreadsheet template.
