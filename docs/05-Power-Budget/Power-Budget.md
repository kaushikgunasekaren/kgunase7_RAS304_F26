---
title: Power Budget
---

# Power Budget

> **Update (23 Nov 2026):** this page shows the design as submitted. In [BOM revision 2](../03-BOM/BOM.md#revision-2-changes-2026-11-23) the motor became a lab Pololu #3067 / DORHEA N20 6 V gearmotor (no lead screw), the slide pot became a 10k rotary pot (P-3), and the IMU became an off-board ADXL345 module on I2C (U5 not fitted). The circuit and PCB are otherwise unchanged.


**RAS 304 – Individual Datasheet (Kaushik) – Driver Nano 3: Linear Actuator B + IMU**

> This budget matches my current [schematic](../04-Schematic/schematic.md): a 6 V motor rail (LM317TG set to 5.9 V) and a separate 5 V logic rail (L7805CV). It was built in the course power budget template. Values marked "est." should be replaced with measured values once the board is built.

**Spreadsheet:** [Power-Budget-KG.xlsx](Power-Budget-KG.xlsx) (course template, with formulas) · [ZIP of the spreadsheet](Power-Budget-KG.zip)

## Power architecture

- **Power is supplied individually.** Each teammate's board has its own adapter, and the ribbon to the hub carries only UART communication and a ground reference, so no other board draws from this budget and this board draws from no other.
- One external source: my own 9 V 1 A wall adapter on the barrel jack, through a removable 1 A fuse and a 1N5819 reverse-polarity diode. About 8.6 V reaches the regulators.
- **+6 V motor rail (LM317TG, set to 5.9 V by R17/R18):** powers only the DRV8833 motor supply (VM) and the N20 gearmotor, which is rated 6 V.
- **+5 V logic rail (L7805CV):** powers the Curiosity Nano (VBUS, through JP1 and D3) and the green power LED.
- **+3.3 V rail (Curiosity Nano on-board regulator, VTG):** powers the ADXL343, the slide potentiometer and the MCP601 current-sense amplifier. It draws from the 5 V rail, so its total is counted there too.

Splitting the motor onto its own regulator keeps motor current spikes and stall current off the rail that powers the microcontroller.

## Section A. Components that consume power (absolute maximum current)

Regulators, the power source, switches, the fuse, LEDs and passives are not listed, following the template.

| Component | Part number | Supply | Max current (mA) | Basis |
|---|---|---|---|---|
| N20 gearmotor with M3 lead screw (Actuator B) | DFRobot FIT0577 | +6 V | 420 | Datasheet stall current at 6 V |
| Dual H-bridge driver (own VM supply current only) | TI DRV8833PWPR | +6 V | 5 | Allowance (est.), above the datasheet operating current |
| PIC18F57Q43 Curiosity Nano (MCU and on-board debugger) | Microchip DM164150 | +5 V | 100 | Conservative allowance (est.) |
| 3-axis accelerometer | Analog Devices ADXL343BCCZ | +3.3 V | 0.5 | Allowance (est.); datasheet typical is about 0.14 mA in measurement mode |
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
| **+3.3 V** | ADXL343BCCZ | 0.5 |
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
| +6 V | onsemi LM317TG (TO-220), set to 5.9 V | 1500 | 531.3 | **968.7** | Lets the 6 V motor run at its rated voltage. Needs at least 8 V in; about 8.6 V is available after the diode |
| +5 V | ST L7805CV (TO-220) | 1500 | 127.1 | **1372.9** | Required 5 V linear regulator; now carries only logic, so it runs cool |
| +3.3 V | Curiosity Nano on-board regulator (VTG) | 500 | 1.7 | **498.3** | Already on the Nano; the sensor load is tiny |

Regulator thermal check (40 °C ambient, 125 °C junction limit, no heatsink: about 50 °C/W for a TO-220 in free air):

| Regulator | Dissipation at 9 V adapter (8.6 V in) | Junction temperature | Heatsink? |
|---|---|---|---|
| LM317TG | (8.6 − 5.9) V × 0.531 A ≈ 1.43 W | ≈ 112 °C | Not strictly needed, but close to the limit, so it gets a small clip-on TO-220 heatsink from TECH 162 as cheap insurance |
| L7805CV | (8.6 − 5) V × 0.127 A ≈ 0.46 W | ≈ 63 °C | No |

If a 12 V adapter is used, the LM317TG dissipates about (11.6 − 5.9) V × 0.531 A ≈ 3.0 W and **needs** a heatsink of about 23 °C/W or better. These worst cases assume the motor is stalled; in normal motion the motor draws much less (24 mA no load).

## Section D. External power source

| Item | Current (mA) |
|---|---|
| LM317TG input: 531.3 + 5.7 (R17/R18 divider; the LM317 has no ground current) | 537.0 |
| L7805CV input: 127.1 + 8 (quiescent) | 135.1 |
| **Total required from the adapter** | **672.1** |
| Adapter capacity (9 V, 1 A) | 1000 |
| **Total remaining current available** | **327.9** |

**Fault case:** the 0.33 Ω DRV8833 sense resistor limits the motor to about 0.61 A. With the motor at that limit, the 6 V rail needs (610 + 5) × 1.25 ≈ 769 mA, and the adapter supplies 769 + 5.7 + 135.1 ≈ 910 mA, so about 90 mA still remains. The 1 A fuse is not reached even in a fault.

## Section E. Battery life

Not applicable. The board runs from a wall adapter and has no battery.

## What the power budget showed

- **It drove the move to a separate motor rail.** The first version put the motor on the 5 V rail. The budget showed the L7805CV would dissipate about 2.6 W and needed a heatsink, and the 6 V motor would run under-voltage. Moving the motor to a 6 V regulator (now the MC7806BTG) fixed both: the 5 V regulator now dissipates under 0.5 W, and the motor gets its rated 6 V.
- **It set the current-limit resistor.** The original 0.2 Ω sense resistor allowed about 1 A in a motor fault, which would exceed the 1 A adapter. The budget showed a limit near 0.6 A is needed, which is why the schematic uses 0.33 Ω (0.61 A).
- **It confirmed the 9 V 1 A adapter is enough.** About 328 mA remains in normal operation and about 90 mA in a fault, so no new adapter has to be bought.
- **It confirmed the 9 V minimum.** The LM317TG needs about 2–3 V of headroom, so about 8 V in, so the adapter must be at least 9 V after the fuse and diode.

## Open items

1. Measure the N20 stall current at 6 V and compare it with the 420 mA datasheet value.
2. Replace the Curiosity Nano, DRV8833 and ADXL343 estimates with measured current.
3. Check the LM317TG heatsink temperature once the board's operating temperature is measured.
