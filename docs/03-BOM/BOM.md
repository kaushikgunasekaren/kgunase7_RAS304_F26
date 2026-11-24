---
title: Bill of Materials
tags:
- BOM
---

## Overview

This is the bill of materials for my individual board, PCB-4 (Driver Nano 3: Motor-2 + IMU + potentiometer P-3). It lists every part on the [schematic](../04-Schematic/schematic.md), using the schematic's reference designators, plus the off-board motor and potentiometer. The tables are sorted by vendor, and each vendor below gets one purchase request.

**Order quantities.** Following the assignment instructions, for every part I order one for the breadboard prototype, one for the subsystem PCB, and one or two spares:

* Through-hole parts (regulator, op-amp, barrel jack, capacitor, sense resistor): 3 each (breadboard, PCB, 1 spare). For the diodes it is 6 on the breadboard + 6 on the PCB + 1 spare = 13.
* The DRV8833 (HTSSOP-16) has no through-hole version, and I am not using a breakout board, so it cannot go on the breadboard. I order 2: one for the PCB and one spare.
* The 5 V regulator is not ordered, because TECH 162 stocks the LM7805T (same TO-220 pinout as the L7805CV).
* Off-board parts (motor, potentiometer, IMU module) come in multi-packs from Amazon that also cover my teammates' boards.
* Parts that TECH 162 stocks (fuses, IC sockets, headers, and common passives, LEDs and buttons) are not ordered. They are listed below with the quantity I will take.

This is more than one board's worth of parts. One finished board uses about $14.78 of purchased parts: $8.55 of DigiKey parts, one potentiometer from the 10-pack (about $0.73) and one ADXL345 module from the 2-pack ($5.50). The motor comes from lab stock.

**Revision 3 (2026-11-24).** The total order has to stay under $60. Using the class quantity rule exactly (above), the DigiKey order drops to $20.91: the DRV8833 goes to 2 and the diodes to 13, and the L7805CV comes from TECH 162 stock. The M3 heat-set inserts ($8.98) were removed: they are a general lab supply, not a part on the schematic, and come from the lab or a teammate. **New total: $52.18.** The ADXL345 (GY-291) is a breakout module, so the order email asks the professor to approve it, as the class page requires.

**Revision 2 (2026-11-23).** The order was redone to match the team's arm sketch and the parts the team agreed on, and so that everything arrives within 2 days. See [Revision 2 changes](#revision-2-changes-2026-11-23) below. DRV8833, 1N5819, LM317TG and PJ-002A prices were rechecked on DigiKey on 2026-11-24 (the MCP601, 2.2 µF capacitor and 0.33 Ω resistor prices are from 2026-11-03), and Amazon prices are from 2026-11-24, when all three Amazon items showed Prime next-day delivery. Shipping and tax are not included.

Datasheets are stored in this repository in the `docs/03-BOM/datasheets/` folder and linked in each row. Where a manufacturer site would not allow the PDF to be saved, the datasheet link goes to the manufacturer or vendor page instead.

## Bill of Materials

### DigiKey (purchase request 1, 2-day shipping)

| Part Name/Description | Qty | Unit Cost | Total Cost | Manufacturer | Manufacturer # | Vendor Link | Datasheet Link | Schematic Reference Designators |
|:--|:--|:--|:--|:--|:--|:--|:--|:--|
| Dual H-bridge motor driver, HTSSOP-16 | 2 | $3.020 | $6.04 | Texas Instruments | DRV8833PWPR | [DigiKey 296-29434-1-ND](https://www.digikey.com/en/products/detail/texas-instruments/DRV8833PWPR/2743167) | [DRV8833.pdf](datasheets/DRV8833.pdf) | U3 |
| Schottky diode 40 V 1 A, DO-41 | 13 | $0.343 | $4.46 | onsemi | 1N5819RLG | [DigiKey 1N5819RLGOSCT-ND](https://www.digikey.com/en/products/detail/onsemi/1N5819RLG/918061) | [1N5819.pdf](datasheets/1N5819.pdf) | D1, D3, D6–D9 |
| Adjustable 1.5 A linear regulator (set to 5.9 V), TO-220 | 3 | $1.300 | $3.90 | onsemi | LM317TG | [DigiKey LM317TGOS-ND](https://www.digikey.com/en/products/detail/onsemi/LM317TG/918508) | [LM317.pdf](datasheets/LM317.pdf) | U6 |
| DC barrel jack, 2.1 mm, PCB mount | 3 | $0.660 | $1.98 | Same Sky (CUI Devices) | PJ-002A | [DigiKey CP-002A-ND](https://www.digikey.com/en/products/detail/same-sky-formerly-cui-devices/PJ-002A/96962) | [PJ-002A.pdf](datasheets/PJ-002A.pdf) | J1 |
| Op-amp (current-sense amplifier), PDIP-8 | 3 | $0.630 | $1.89 | Microchip | MCP601-I/P | [DigiKey MCP601-I/P-ND](https://www.digikey.com/en/products/result?keywords=MCP601-I/P) | [MCP601.pdf](datasheets/MCP601.pdf) | U4 |
| Ceramic capacitor 2.2 µF 25 V X7R, radial | 3 | $0.490 | $1.47 | TDK | FA16X7R1E225KNU06 | [DigiKey 445-180433-1-ND](https://www.digikey.com/en/products/result?keywords=FA16X7R1E225KNU06) | [DigiKey page](https://www.digikey.com/en/products/result?keywords=FA16X7R1E225KNU06) | C11 |
| Current-sense resistor 0.33 Ω 1 W 5%, axial | 3 | $0.390 | $1.17 | YAGEO | FMP100JR-52-0R33 | [DigiKey 0.33WCT-ND](https://www.digikey.com/en/products/result?keywords=FMP100JR-52-0R33) | [YAGEO page](https://www.yageo.com/en/ProductSearch/PartNumber?part_number=FMP100JR-52-0R33) | R8 |
| **DigiKey subtotal** | | | **$20.91** | | | | | |

DigiKey is the only non-Amazon vendor. The DRV8833 is a surface-mount IC that the PCB footprint needs, and Amazon sells it only on breakout boards. The other DigiKey parts ship in the same order.

### Amazon (purchase request 2, Prime 1–2 day delivery)

| Part Name/Description | Qty | Unit Cost | Total Cost | Manufacturer | Manufacturer # | Vendor Link | Datasheet Link | Schematic Reference Designators |
|:--|:--|:--|:--|:--|:--|:--|:--|:--|
| N20 6 V 200 RPM gearmotor, 4-pack (164 RPM rated, 0.6 A stall, D-shaft; Motor-2, off-board, on J4) | 1 | $12.99 | $12.99 | DORHEA | ASIN B0HB558JTS | [Amazon B0HB558JTS](https://www.amazon.com/dp/B0HB558JTS) | [Amazon page](https://www.amazon.com/dp/B0HB558JTS) | J4 (off-board) |
| WH148 B10K linear rotary potentiometer, 10-pack (P-1 to P-3; mine is P-3, off-board, on J7) | 1 | $7.29 | $7.29 | Comimark | ASIN B07WWLC33K | [Amazon B07WWLC33K](https://www.amazon.com/dp/B07WWLC33K) | [Amazon page](https://www.amazon.com/dp/B07WWLC33K) | J7 (off-board) |
| ADXL345 (GY-291) 3-axis accelerometer module, 2-pack (on the arm link, I2C to J6) | 1 | $10.99 | $10.99 | Hailege | ASIN B08D7F38XY | [Amazon B08D7F38XY](https://www.amazon.com/dp/B08D7F38XY) | [ADXL345 datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/adxl345.pdf) | J6-10 to J6-13, J5-2 (off-board) |
| **Amazon subtotal** | | | **$31.27** | | | | | |

One 4-pack of motors gives the 3 extra N20 motors the team asked for, plus a spare. The 10-pack of potentiometers covers P-1 (PCB-2), P-2 (PCB-3) and P-3 (mine).

### TECH 162 lab stock and course kit (not ordered)

The assignment lists fuses, IC sockets and headers as TECH 162 stock. If TECH 162 runs out of any part here, it is a common, in-stock DigiKey part.


| Part Name/Description | Qty | Unit Cost | Total Cost | Manufacturer | Manufacturer # | Vendor Link | Datasheet Link | Schematic Reference Designators |
|:--|:--|:--|:--|:--|:--|:--|:--|:--|
| N20 micro metal gearmotor 210:1 HPCB 6 V, 160 RPM, 3 mm D-shaft (Motor-2) | 1 | lab stock | $0 | Pololu | 3067 | lab stock | [Pololu page](https://www.pololu.com/product/3067) | J4 (off-board) |
| 5 V 1.5 A linear regulator, TO-220 (TECH 162 stocks the LM7805T; same pinout as the L7805CV) | 3 | TECH 162 | $0 | STMicroelectronics / lab LM7805T | L7805CV | TECH 162 | [L78xx.pdf](datasheets/L78xx.pdf) | U2 |
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

Test points TP1–TP13 are bare copper pads on the PCB, so they cost nothing. JP1 and JP2 are 2-pin jumpers that let me disconnect the Nano's VBUS feed and the motor rail while I debug. Dupont jumper wire for the off-board IMU cable also comes from lab stock, and the M3 heat-set inserts for the 3D-printed links come from the lab or a teammate.

**Not fitted:** U5 (ADXL343BCCZ) is marked DNP (do not populate) on the schematic and PCB. Its pads stay on the board, but it is not ordered (see below).

## Revision 2 changes (2026-11-23)

The team's arm sketch gives PCB-4 (my board) **Motor-2, the IMU and potentiometer P-3**, reporting to PCB-1 (Preston, IK and FK). The circuit and the PCB copper are unchanged: the new motor and potentiometer plug into the same headers (J4, J7), and the IMU moves off the board.

| Removed | Why | Replaced by |
|:--|:--|:--|
| DFRobot FIT0577 lead-screw motor ×3 ($27.00) | The lab has Pololu #3067 N20 motors, and DFRobot ships from China, too slowly for the 2-day rule | Lab Pololu #3067, plus a DORHEA N20 4-pack from Amazon for the 3 extra motors the team asked for |
| Bourns PTA2043 slide pot ×3 ($5.34) | The joint is now a rotary hinge, and the team chose a rotary pot | Comimark WH148 B10K rotary pot (10-pack, shared with P-1 and P-2) on the same 3-pin header J7 |
| ADXL343BCCZ ×3 ($12.06) | The sketch puts the IMU on the arm link, not on the board | Hailege ADXL345 (GY-291) module on the arm link, wired to the spare headers; U5 marked DNP |

**Motor current.** The 0.33 Ω sense resistor still makes the DRV8833 limit the motor current to about 0.61 A. The Amazon motor stalls at 0.6 A, right at that limit; the lab Pololu #3067 stalls at 1.5 A, so the driver limits it to 0.61 A, which costs some torque but protects the motor. The 1 A fuse stays.

**Off-board IMU wiring** (GY-291 to PCB-4 spare headers; keep the cable under about 30 cm):

| GY-291 pin | PCB-4 header | Nano pin | Function |
|:--|:--|:--|:--|
| VCC | J6-12 | +3V3 | power |
| GND | J6-13 | GND | ground |
| SDA | J6-11 | RC0 | I2C1 SDA (via PPS) |
| SCL | J6-10 | RC1 | I2C1 SCL (via PPS) |
| INT1 (optional) | J5-2 | RB2 | INT0 / INT2 (via PPS) |

On the 48-pin PIC18F57Q43, I2C1 SDA and SCL can be mapped to ports B and C, and INT0 to ports A and B (PPS tables, datasheet DS40002147E). RC0 and RC1 are free on the Curiosity Nano because its 32.768 kHz crystal footprint is not connected by default. The firmware maps I2C1 to RC0/RC1 with open-drain outputs at 100 kHz; the module's address is 0x53. The ADXL345 uses the same registers as the ADXL343, so the IMU code barely changes.

**Still to confirm with the team:** the sketch labels the motor "DF Robot DC", but the later team message switched to the lab's N20 motors, which have no built-in lead screw. The hinge drive has to come from the mechanical design.

## Changes from the original design (2026-11-03)

The [schematic](../04-Schematic/schematic.md) has been updated for all of these, so the reference designators and part numbers above match it.

* **U5:** the ST LIS3DHTR was out of stock (DigiKey has none until March 2027, and the ST store and other distributors I checked had none). It is replaced by the Analog Devices ADXL343BCCZ, the lower-cost, pin-compatible version of the ADXL345 that was my second choice on the [Component Selection](../02-Component-Selection/Component-Selection.md) page. U5 is rewired for its pinout on the same SPI1, chip-select and INT1 nets.
* **U6:** every fixed 6 V 78xx regulator I checked is obsolete (ST L7806CV, L7806ABV, L7806ACV; onsemi MC7806BTG, MC7806CTG, MC78M06CTG), and DigiKey sells the onsemi ones only through a third-party marketplace. U6 is now an onsemi LM317TG adjustable regulator, which is active and in stock. R17 (220 Ω) and R18 (820 Ω) set it to 5.9 V. It gets a small heatsink (see the [Power Budget](../05-Power-Budget/Power-Budget.md)).
* **R8:** the Ohmite 41FR33E is not stocked at DigiKey (minimum 250, 12-week lead time). The YAGEO FMP100JR-52-0R33 has the same value, rating, and axial footprint.
* **C11:** the KEMET C320C225K3R5TA is not stocked (minimum 2,000). The TDK FA16X7R1E225KNU06 has the same value, voltage, and dielectric, with 2.5 mm lead spacing, which fits the footprint.
* **C2, C9, C17:** the Nichicon UVR parts are obsolete or unavailable. These are common values that I will take from TECH 162.

## Cost summary

| Vendor | Items | Subtotal |
|:--|:--|:--|
| DigiKey | 7 line items | $20.91 |
| Amazon | 3 line items | $31.27 |
| **Total new purchases** | | **$52.18** |

The order limit is under $60. The order covers a breadboard prototype, a PCB and spares, as the assignment asks, plus multi-packs shared with teammates. One finished board uses about $14.78 of these parts. DigiKey 2-day shipping and tax are extra.

The KiCad project, with U5 marked DNP, is in the [PCB Design ZIP](../06-PCB-Design/Subsystem-PCB-KG-ECAD.zip). The revision 3 table, with a status column that marks each line as ordered, removed or lab stock, is saved as [BOM-KG.csv](BOM-KG.csv).
