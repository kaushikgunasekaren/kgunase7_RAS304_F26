---
title: Power Budget
---

# Power Budget

**RAS 304 – Individual Datasheet (Kaushik) – Driver Nano 3: Linear Actuator B + IMU**

> This budget matches my current [schematic](../04-Schematic/schematic.md): a 6 V motor rail (L7806CV) and a separate 5 V logic rail (L7805CV). It was built in the course power budget template. Values marked "est." should be replaced with measured values once the board is built.

**Spreadsheet:** [Power-Budget-KG.xlsx](Power-Budget-KG.xlsx) (course template, with formulas) · [ZIP of the spreadsheet](Power-Budget-KG.zip)

## Power architecture

- One external source: my own 9 V 1 A wall adapter on the barrel jack, through a removable 1 A fuse and a 1N5819 reverse-polarity diode. About 8.6 V reaches the regulators.
- **+6 V motor rail (L7806CV):** powers only the DRV8833 motor supply (VM) and the N20 gearmotor, which is rated 6 V.
- **+5 V logic rail (L7805CV):** powers the Curiosity Nano (VBUS, through JP1 and D3) and the green power LED.
- **+3.3 V rail (Curiosity Nano on-board regulator, VTG):** powers the LIS3DH, the slide potentiometer and the MCP601 current-sense amplifier. It draws from the 5 V rail, so its total is counted there too.

Splitting the motor onto its own regulator keeps motor current spikes and stall current off the rail that powers the microcontroller.

## Section A. Components that consume power (absolute maximum current)

Regulators, the power source, switches, the fuse, LEDs and passives are not listed, following the template.

| Component | Part number | Supply | Max current (mA) | Basis |
|---|---|---|---|---|
| N20 gearmotor with M3 lead screw (Actuator B) | DFRobot FIT0577 | +6 V | 420 | Datasheet stall current at 6 V |
| Dual H-bridge driver (own VM supply current only) | TI DRV8833PWPR | +6 V | 5 | Allowance (est.), above the datasheet operating current |
| PIC18F57Q43 Curiosity Nano (MCU and on-board debugger) | Microchip DM164150 | +5 V | 100 | Conservative allowance (est.) |
| 3-axis accelerometer | ST LIS3DHTR | +3.3 V | 0.5 | Allowance (est.) for operating mode |
| Slide potentiometer, 10 kΩ ±20% | Bourns PTA2043-2015CPB103 | +3.3 V | 0.5 | 3.3 V ÷ 8 kΩ (worst-case low resistance) ≈ 0.41 mA, rounded up |
| Op-amp (current-sense amplifier) | Microchip MCP601-I/P | +3.3 V | 0.325 | Datasheet maximum quiescent current |

The DRV8833 does not itself draw the motor current; the motor does, so it is not counted twice.

## Section B. Rail totals with 25% safety margin

| Rail | Component | Current (mA) |
|---|---|---|
| **+6 V** | N20 gearmotor (stall) | 420 |
| +6 V | DRV8833 VM supply | 5 |
| | Subtotal | 425 |
| | **Required (× 1.25)** | **531.3** |
| **+3.3 V** | LIS3DHTR | 0.5 |
| +3.3 V | Slide potentiometer + MCP601 | 0.825 |
| | Subtotal | 1.325 |
| | **Required (× 1.25)** | **1.7** |
| **+5 V** | Curiosity Nano | 100 |
| +5 V | +3.3 V rail (Nano regulator input) | 1.7 |
| | Subtotal | 101.7 |
| | **Required (× 1.25)** | **127.1** |

## Section C. Regulator per rail

| Rail | Regulator | Rated (mA) | Required (mA) | Remaining (mA) | Rationale |
|---|---|---|---|---|---|
| +6 V | ST L7806CV (TO-220) | 1500 | 531.3 | **968.8** | Lets the 6 V motor run at its rated voltage. Needs at least 8 V in; about 8.6 V is available after the diode |
| +5 V | ST L7805CV (TO-220) | 1500 | 127.1 | **1372.9** | Required 5 V linear regulator; now carries only logic, so it runs cool |
| +3.3 V | Curiosity Nano on-board regulator (VTG) | 500 | 1.7 | **498.3** | Already on the Nano; the sensor load is tiny |

Regulator thermal check (TO-220, about 50 °C/W with no heatsink, 40 °C ambient, 125 °C junction limit):

| Regulator | Dissipation at 9 V adapter (8.6 V in) | Junction temperature | Heatsink? |
|---|---|---|---|
| L7806CV | (8.6 − 6) V × 0.531 A ≈ 1.38 W | ≈ 109 °C | Not strictly needed, but close to the limit; a small clip-on heatsink is cheap insurance |
| L7805CV | (8.6 − 5) V × 0.127 A ≈ 0.46 W | ≈ 63 °C | No |

If a 12 V adapter is used, the L7806CV dissipates about (11.6 − 6) V × 0.531 A ≈ 3.0 W and **needs** a heatsink of about 23 °C/W or better. These worst cases assume the motor is stalled; in normal motion the motor draws much less (24 mA no load).

## Section D. External power source

| Item | Current (mA) |
|---|---|
| L7806CV input: 531.3 + 8 (quiescent) | 539.3 |
| L7805CV input: 127.1 + 8 (quiescent) | 135.1 |
| **Total required from the adapter** | **674.3** |
| Adapter capacity (9 V, 1 A) | 1000 |
| **Total remaining current available** | **325.7** |

**Fault case:** the 0.33 Ω DRV8833 sense resistor limits the motor to about 0.61 A. With the motor at that limit, the 6 V rail needs (610 + 5) × 1.25 ≈ 769 mA, and the adapter supplies 769 + 8 + 135.1 ≈ 912 mA, so about 88 mA still remains. The 1 A fuse is not reached even in a fault.

## Section E. Battery life

Not applicable. The board runs from a wall adapter and has no battery.

## What the power budget showed

- **It drove the move to a separate motor rail.** The first version put the motor on the 5 V rail. The budget showed the L7805CV would dissipate about 2.6 W and needed a heatsink, and the 6 V motor would run under-voltage. Moving the motor to an L7806CV fixed both: the 5 V regulator now dissipates under 0.5 W, and the motor gets its rated 6 V.
- **It set the current-limit resistor.** The original 0.2 Ω sense resistor allowed about 1 A in a motor fault, which would exceed the 1 A adapter. The budget showed a limit near 0.6 A is needed, which is why the schematic uses 0.33 Ω (0.61 A).
- **It confirmed the 9 V 1 A adapter is enough.** About 326 mA remains in normal operation and about 88 mA in a fault, so no new adapter has to be bought.
- **It confirmed the 9 V minimum.** The L7806CV needs about 8 V in, so the adapter must be at least 9 V after the fuse and diode.

## Open items

1. Measure the N20 stall current at 6 V and compare it with the 420 mA datasheet value.
2. Replace the Curiosity Nano, DRV8833 and LIS3DH estimates with measured current.
3. Decide on the optional L7806CV heatsink once the board's operating temperature is measured.
