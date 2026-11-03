---
title: Bill of Materials
tags:
- BOM
---

## Overview

This is the bill of materials for my individual board (Driver Nano 3: Linear Actuator B + IMU). It lists every part on the [schematic](../04-Schematic/schematic.md), using the schematic's reference designators, plus the off-board motor and potentiometer. The tables are sorted by vendor, and each vendor below gets one purchase request.

**Order quantities.** Following the assignment instructions, for every part I order one for the breadboard prototype, one for the subsystem PCB, and one or two spares:

* Through-hole parts (regulators, op-amp, barrel jack, diodes), the off-board motor and the slide potentiometer: 3 each (breadboard, PCB, spare). For the diodes it is 2 boards × 6 + 3 spares = 15.
* The two surface-mount ICs (DRV8833 in HTSSOP-16 and ADXL343 in LGA-14) have no through-hole version, and I am not using breakout boards, so they cannot go on the breadboard. I order 3 of each: one for the PCB and two spares, because these are the hardest parts to solder.
* Parts that TECH 162 stocks (fuses, IC sockets, headers, and common passives, LEDs and buttons) are not ordered. They are listed below with the quantity I will take.

This is more than one board's worth of parts. One finished board uses about $24.49 of purchased parts (see [Component Selection, section 4](../02-Component-Selection/Component-Selection.md)).

Prices and stock are from DigiKey and DFRobot, checked on 2026-11-03. Every ordered part was in stock and active that day, and nothing is back-ordered or special-order. Shipping and tax are not included.

Datasheets are stored in this repository in the `docs/03-BOM/datasheets/` folder and linked in each row. Where a manufacturer site would not allow the PDF to be saved, the datasheet link goes to the manufacturer or vendor page instead.

## Bill of Materials

### DigiKey (purchase request 1)

| Part Name/Description | Qty | Unit Cost | Total Cost | Manufacturer | Manufacturer # | Vendor Link | Datasheet Link | Schematic Reference Designators |
|:--|:--|:--|:--|:--|:--|:--|:--|:--|
| 3-axis accelerometer, SPI/I2C, LGA-14 (3 × 5 mm) | 3 | $4.020 | $12.06 | Analog Devices | ADXL343BCCZ | [DigiKey 505-ADXL343BCCZ-ND](https://www.digikey.com/en/products/detail/analog-devices-inc/ADXL343BCCZ/3542918) | [ADXL343.pdf](datasheets/ADXL343.pdf) | U5 |
| Dual H-bridge motor driver, HTSSOP-16 | 3 | $3.020 | $9.06 | Texas Instruments | DRV8833PWPR | [DigiKey 296-29434-1-ND](https://www.digikey.com/en/products/detail/texas-instruments/DRV8833PWPR/2743167) | [DRV8833.pdf](datasheets/DRV8833.pdf) | U3 |
| Slide potentiometer 10 kΩ, 20 mm travel (off-board, on J7) | 3 | $1.780 | $5.34 | Bourns | PTA2043-2015CPB103 | [DigiKey PTA2043-2015CPB103-ND](https://www.digikey.com/en/products/detail/bourns-inc/PTA2043-2015CPB103/3781176) | [PTA-slide-pot.pdf](datasheets/PTA-slide-pot.pdf) | – |
| Schottky diode 40 V 1 A, DO-41 | 15 | $0.343 | $5.15 | onsemi | 1N5819RLG | [DigiKey 1N5819RLGOSCT-ND](https://www.digikey.com/en/products/detail/onsemi/1N5819RLG/918061) | [1N5819.pdf](datasheets/1N5819.pdf) | D1, D3, D6–D9 |
| Adjustable 1.5 A linear regulator (set to 5.9 V), TO-220 | 3 | $1.300 | $3.90 | onsemi | LM317TG | [DigiKey LM317TGOS-ND](https://www.digikey.com/en/products/result?keywords=LM317TG) | [LM317.pdf](datasheets/LM317.pdf) | U6 |
| 5 V 1.5 A linear regulator, TO-220 | 3 | $1.140 | $3.42 | STMicroelectronics | L7805CV | [DigiKey 497-1443-5-ND](https://www.digikey.com/en/products/detail/stmicroelectronics/L7805CV/585964) | [L78xx.pdf](datasheets/L78xx.pdf) | U2 |
| DC barrel jack, 2.1 mm, PCB mount | 3 | $0.660 | $1.98 | Same Sky (CUI Devices) | PJ-002A | [DigiKey CP-002A-ND](https://www.digikey.com/en/products/detail/same-sky-formerly-cui-devices/PJ-002A/96962) | [PJ-002A.pdf](datasheets/PJ-002A.pdf) | J1 |
| Op-amp (current-sense amplifier), PDIP-8 | 3 | $0.630 | $1.89 | Microchip | MCP601-I/P | [DigiKey MCP601-I/P-ND](https://www.digikey.com/en/products/result?keywords=MCP601-I/P) | [MCP601.pdf](datasheets/MCP601.pdf) | U4 |
| Ceramic capacitor 2.2 µF 25 V X7R, radial | 3 | $0.490 | $1.47 | TDK | FA16X7R1E225KNU06 | [DigiKey 445-180433-1-ND](https://www.digikey.com/en/products/result?keywords=FA16X7R1E225KNU06) | [DigiKey page](https://www.digikey.com/en/products/result?keywords=FA16X7R1E225KNU06) | C11 |
| Current-sense resistor 0.33 Ω 1 W 5%, axial | 3 | $0.390 | $1.17 | YAGEO | FMP100JR-52-0R33 | [DigiKey 0.33WCT-ND](https://www.digikey.com/en/products/result?keywords=FMP100JR-52-0R33) | [YAGEO page](https://www.yageo.com/en/ProductSearch/PartNumber?part_number=FMP100JR-52-0R33) | R8 |
| **DigiKey subtotal** | | | **$45.44** | | | | | |

### DFRobot (purchase request 2)

| Part Name/Description | Qty | Unit Cost | Total Cost | Manufacturer | Manufacturer # | Vendor Link | Datasheet Link | Schematic Reference Designators |
|:--|:--|:--|:--|:--|:--|:--|:--|:--|
| N20 6 V 98 RPM gearmotor with M3 × 20 mm lead screw (off-board, on J4) | 3 | $9.000 | $27.00 | DFRobot | FIT0577 | [DFRobot FIT0577](https://www.dfrobot.com/product-1733.html) | [DFRobot page](https://www.dfrobot.com/product-1733.html) | – |
| **DFRobot subtotal** | | | **$27.00** | | | | | |

No vendor on the course list had this motor in stock: RobotShop lists it as special order, and DigiKey only as a non-stock part with a 9-week lead time. The course sourcing guide allows another supplier when the preferred vendors are out of stock, so it comes directly from DFRobot's US store, which showed it in stock. DFRobot ships from China, so shipping and possible import duty are extra.

### TECH 162 lab stock and course kit (not ordered)

The assignment lists fuses, IC sockets and headers as TECH 162 stock. If TECH 162 runs out of any part here, it is a common, in-stock DigiKey part.


| Part Name/Description | Qty | Unit Cost | Total Cost | Manufacturer | Manufacturer # | Vendor Link | Datasheet Link | Schematic Reference Designators |
|:--|:--|:--|:--|:--|:--|:--|:--|:--|
| PIC18F57Q43 Curiosity Nano | 1 | course kit | $0 | Microchip | DM164150 | course kit | [Microchip page](https://www.microchip.com/en-us/development-tool/DM164150) | U1 |
| 9 V 1 A wall adapter, 5.5 × 2.1 mm, center-positive | 1 | already owned | $0 | – | – | – | [Adafruit page](https://www.adafruit.com/product/63) | – |
| Fuse 1 A fast-acting, 5 × 20 mm | 5 | TECH 162 | $0 | Littelfuse | 0217001.HXP | TECH 162 | [DigiKey page](https://www.digikey.com/en/products/detail/littelfuse-inc/0217001-HXP/777108) | F1 |
| Fuse clip, 5 mm, PCB mount | 4 | TECH 162 | $0 | Keystone | 3517 | TECH 162 | [Keystone-3517.pdf](datasheets/Keystone-3517.pdf) | F1 |
| DIP-8 IC socket (for the op-amp) | 3 | TECH 162 | $0 | any | – | TECH 162 | – | U4 |
| TO-220 clip-on heatsink (for the 6 V regulator) | 2 | TECH 162 | $0 | any | – | TECH 162 | – | U6 |
| 2x4 IDC shrouded header (ribbon to hub) | 3 | TECH 162 | $0 | Molex | 0702460801 | TECH 162 | [DigiKey page](https://www.digikey.com/en/products/result?keywords=0702460801) | J2 |
| Break-away male header strip, 2.54 mm | 2 | TECH 162 | $0 | Sullins | PREC040SAAN-RC | TECH 162 | – | J3–J7, JP1, JP2 |
| Jumper shunt, 2.54 mm | 4 | TECH 162 | $0 | Sullins | QPC02SXGN-RC | TECH 162 | – | JP1, JP2 |
| Female header 1x28, 2.54 mm (Curiosity Nano socket) | 4 | TECH 162 | $0 | Sullins | PPTC281LFBN-RC | TECH 162 | – | U1 |
| Tactile push button, 6 mm | 3 | TECH 162 | $0 | TE Connectivity | 1825910-6 | TECH 162 | [TE-1825910-switch.pdf](datasheets/TE-1825910-switch.pdf) | SW1 |
| LED 5 mm green / red / yellow | 3 each | TECH 162 | $0 | Kingbright | WP7113GD / WP7113ID / WP7113YD | TECH 162 | [WP7113-LED.pdf](datasheets/WP7113-LED.pdf) | D2, D4, D5 |
| Ceramic capacitor 0.33 µF 50 V | 3 | TECH 162 | $0 | KEMET | C320C334K5R5TA | TECH 162 | [KEMET page](https://www.kemet.com/en/us/capacitors/ceramic/products/goldmax.html) | C1 |
| Ceramic capacitor 0.1 µF 50 V | 26 | TECH 162 | $0 | KEMET | C320C104K5R5TA | TECH 162 | [KEMET page](https://www.kemet.com/en/us/capacitors/ceramic/products/goldmax.html) | C3–C7, C10, C12–C16, C18 |
| Ceramic capacitor 0.01 µF 50 V | 3 | TECH 162 | $0 | KEMET | C315C103K5R5TA | TECH 162 | [KEMET page](https://www.kemet.com/en/us/capacitors/ceramic/products/goldmax.html) | C8 |
| Electrolytic capacitor 47 µF 25 V, radial | 3 | TECH 162 | $0 | Nichicon | UVR1E470MDD (obsolete; any 47 µF ≥ 16 V radial) | TECH 162 | – | C2 |
| Electrolytic capacitor 10 µF 25 V, radial | 5 | TECH 162 | $0 | Nichicon | UVR1E100MDD (any 10 µF ≥ 16 V radial) | TECH 162 | – | C9, C17 |
| Resistor 1 kΩ 1/4 W | 10 | TECH 162 | $0 | Stackpole | CF14JT1K00 | TECH 162 | – | R1, R5, R13, R14 |
| Resistor 330 Ω 1/4 W | 5 | TECH 162 | $0 | Stackpole | CF14JT330R | TECH 162 | – | R2, R3 |
| Resistor 10 kΩ 1/4 W | 16 | TECH 162 | $0 | Stackpole | CF14JT10K0 | TECH 162 | – | R4, R6, R7, R9, R10, R15, R16 |
| Resistor 47 kΩ 1/4 W | 3 | TECH 162 | $0 | Stackpole | CF14JT47K0 | TECH 162 | – | R11 |
| Resistor 4.7 kΩ 1/4 W | 3 | TECH 162 | $0 | Stackpole | CF14JT4K70 | TECH 162 | – | R12 |
| Resistor 220 Ω 1/4 W (LM317 voltage set) | 3 | TECH 162 | $0 | Stackpole | CF14JT220R | TECH 162 | – | R17 |
| Resistor 820 Ω 1/4 W (LM317 voltage set) | 3 | TECH 162 | $0 | Stackpole | CF14JT820R | TECH 162 | – | R18 |

Test points TP1–TP13 are bare copper pads on the PCB, so they cost nothing. JP1 and JP2 are 2-pin jumpers that let me disconnect the motor rail and the IMU while I debug.

## Changes from the original design

The [schematic](../04-Schematic/schematic.md) has been updated for all of these, so the reference designators and part numbers above match it.

* **U5:** the ST LIS3DHTR was out of stock (DigiKey has none until March 2027, and the ST store and other distributors I checked had none). It is replaced by the Analog Devices ADXL343BCCZ, the lower-cost, pin-compatible version of the ADXL345 that was my second choice on the [Component Selection](../02-Component-Selection/Component-Selection.md) page. U5 is rewired for its pinout on the same SPI1, chip-select and INT1 nets.
* **U6:** every fixed 6 V 78xx regulator I checked is obsolete (ST L7806CV, L7806ABV, L7806ACV; onsemi MC7806BTG, MC7806CTG, MC78M06CTG), and DigiKey sells the onsemi ones only through a third-party marketplace. U6 is now an onsemi LM317TG adjustable regulator, which is active and in stock. R17 (220 Ω) and R18 (820 Ω) set it to 5.9 V. It gets a small heatsink (see the [Power Budget](../05-Power-Budget/Power-Budget.md)).
* **R8:** the Ohmite 41FR33E is not stocked at DigiKey (minimum 250, 12-week lead time). The YAGEO FMP100JR-52-0R33 has the same value, rating, and axial footprint.
* **C11:** the KEMET C320C225K3R5TA is not stocked (minimum 2,000). The TDK FA16X7R1E225KNU06 has the same value, voltage, and dielectric, with 2.5 mm lead spacing, which fits the footprint.
* **C2, C9, C17:** the Nichicon UVR parts are obsolete or unavailable. These are common values that I will take from TECH 162.

## Cost summary

| Vendor | Items | Subtotal |
|:--|:--|:--|
| DigiKey | 10 line items | $45.44 |
| DFRobot | 1 line item | $27.00 |
| **Total new purchases** | | **$72.44** |

The order covers a breadboard prototype, a PCB and spares, as the assignment asks. One finished board uses about $24.49 of these parts. Shipping is extra, and DFRobot may add import duty.

The KiCad BOM export (`BOM.csv`) is inside the [schematic ZIP](../04-Schematic/Subsystem-Schematic-KG.zip). This page's table is also saved as [BOM-KG.csv](BOM-KG.csv).
