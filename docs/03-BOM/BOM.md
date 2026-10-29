---
title: Bill of Materials
tags:
- BOM
---

## Overview

This is the bill of materials for my individual board (Driver Nano 3: Linear Actuator B + IMU). It matches the [schematic](../04-Schematic/schematic.md); the reference designators are the ones on the schematic. Prices are US quantity-1 prices from my [Component Selection](../02-Component-Selection/Component-Selection.md) page where I researched them. Rows marked "est." are estimates, and rows marked "lab stock" are common parts expected to come from the lab. The KiCad BOM export (`BOM.csv`) is inside the [schematic ZIP](../04-Schematic/Subsystem-Schematic-KG.zip).

## Major components

| Part | Qty | Unit cost | Total | Manufacturer | Part number | Datasheet | Ref. designators |
|:--|:--|:--|:--|:--|:--|:--|:--|
| N20 6 V gearmotor with M3 lead screw (off-board, on J4) | 1 | $9.00 | $9.00 | DFRobot | FIT0577 | [product page](https://www.dfrobot.com/product-1733.html) | – |
| Dual H-bridge motor driver, HTSSOP-16 | 1 | $3.02 | $3.02 | Texas Instruments | DRV8833PWPR | [datasheet](https://www.ti.com/lit/ds/symlink/drv8833.pdf) | U3 |
| 3-axis accelerometer, LGA-16 | 1 | $1.94 | $1.94 | STMicroelectronics | LIS3DHTR | [datasheet](https://www.st.com/resource/en/datasheet/lis3dh.pdf) | U5 |
| Slide potentiometer 10 kΩ, 20 mm (off-board, on J7) | 1 | $1.84 | $1.84 | Bourns | PTA2043-2015CPB103 | [product page](https://www.digikey.com/en/products/detail/bourns-inc/PTA2043-2015CPB103/3781176) | – |
| 5 V linear regulator, TO-220 | 1 | $0.65 | $0.65 | STMicroelectronics | L7805CV | [datasheet](https://www.st.com/resource/en/datasheet/l78.pdf) | U2 |
| 6 V linear regulator, TO-220 | 1 | ~$0.65 (est.) | ~$0.65 | STMicroelectronics | L7806CV | [datasheet](https://www.st.com/resource/en/datasheet/l78.pdf) | U6 |
| Op-amp, PDIP-8 | 1 | ~$0.50 (est.) | ~$0.50 | Microchip | MCP601-I/P | [datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/21314g.pdf) | U4 |
| PIC18F57Q43 Curiosity Nano | 1 | course kit | $0 | Microchip | DM164150 | – | U1 |
| 9 V 1 A wall adapter | 1 | already owned | $0 | – | – | – | – |

## Power, protection and connectors

| Part | Qty | Manufacturer | Part number | Ref. designators | Cost |
|:--|:--|:--|:--|:--|:--|
| DC barrel jack | 1 | Same Sky (CUI Devices) | PJ-002A | J1 | ~$1 (est.) |
| 1 A fast-acting fuse + 2 fuse clips | 1 | Littelfuse / Keystone | 0217001.HXP + 3517 | F1 | ~$1 (est.) |
| Schottky diode 40 V 1 A | 6 | onsemi | 1N5819RLG | D1, D3, D6–D9 | ~$0.60 (est.) |
| 2x4 IDC header (ribbon to hub) | 1 | Molex | 0702460801 | J2 | lab stock |
| Break-away header strip (J3–J7, JP1, JP2) + 2 shunts | 1 | Sullins | PREC040SAAN-RC, QPC02SXGN-RC | J3–J7, JP1, JP2 | lab stock |
| Female headers for the Curiosity Nano | 2 | Sullins | PPTC281LFBN-RC | U1 | lab stock |
| Push button | 1 | TE Connectivity | 1825910-6 | SW1 | lab stock |
| LEDs (green, red, yellow) | 3 | Kingbright | WP7113GD / WP7113ID / WP7113YD | D2, D4, D5 | ~$0.30 (est.) |

## Passives

| Part | Qty | Manufacturer | Part number | Ref. designators |
|:--|:--|:--|:--|:--|
| 0.33 µF 50 V ceramic | 1 | KEMET | C320C334K5R5TA | C1 |
| 47 µF 25 V electrolytic | 1 | Nichicon | UVR1E470MDD | C2 |
| 0.1 µF 50 V ceramic | 12 | KEMET | C320C104K5R5TA | C3–C7, C10, C12–C16, C18 |
| 0.01 µF 50 V ceramic | 1 | KEMET | C315C103K5R5TA | C8 |
| 10 µF 25 V electrolytic | 2 | Nichicon | UVR1E100MDD | C9, C17 |
| 2.2 µF 25 V ceramic | 1 | KEMET | C320C225K3R5TA | C11 |
| 0.33 Ω 1 W 1% (current sense) | 1 | Ohmite | 41FR33E | R8 |
| 1 kΩ 1/4 W | 4 | Stackpole | CF14JT1K00 | R1, R5, R13, R14 |
| 330 Ω 1/4 W | 2 | Stackpole | CF14JT330R | R2, R3 |
| 10 kΩ 1/4 W | 7 | Stackpole | CF14JT10K0 | R4, R6, R7, R9, R10, R15, R16 |
| 47 kΩ 1/4 W | 1 | Stackpole | CF14JT47K0 | R11 |
| 4.7 kΩ 1/4 W | 1 | Stackpole | CF14JT4K70 | R12 |

Test pads TP1–TP13 are bare copper on the PCB and cost nothing.

## Cost summary

New purchases for this board come to about **$21.50** before shipping (see [Component Selection, section 4](../02-Component-Selection/Component-Selection.md)), slightly above an even share ($17.50–$20.00) of the team's $70–80 budget. Passives, headers and the button are expected from lab stock; if they must be bought, they add a few dollars.
