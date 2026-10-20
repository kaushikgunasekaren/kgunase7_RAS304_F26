---
title: Component Selection
---

# Component Selection

**RAS 304 – Individual Datasheet (Kaushik) – Driver Nano 3: Linear Actuator B + IMU**
Project: Team 102 Modular Spider Leg (my board is the spoke on hub connector 3)

> Scope: this page covers only my individual board (Driver Nano 3). Team-level decisions are in the team repository. The selected parts are the same ones shown on my [Block Diagram](../01-Block-Diagram/Block-Diagram.md) page. Prices were checked on 2026-10-19 and will change.

## 1. Subsystem summary

Driver Nano 3 handles Linear Actuator B control and IMU sensing. It follows this loop from the team software flow diagram:

1. Receive the actuator B target position from the Main Processor (Preston's hub board).
2. Drive Linear Actuator B (PWM and direction).
3. Read the IMU orientation (I2C / SPI).
4. Check whether the orientation is within the expected range. If yes, continue the motion. If no, adjust the motion or flag a status.
5. Send actuator B status and IMU data back to the Main Processor.
6. Loop, waiting for the next command.

When the Main Processor sends a stop command, Driver Nano 3 disables the driver and reports a safe status. The Main Processor does the kinematics; my board only runs local control and returns feedback continuously. The team calls the sensor output "jerk / impulse" data, which the PIC computes from the accelerometer readings.

**Budget constraint:** the whole team budget is $70 to $80 for four boards, so each board should cost about $17.50 to $20.00. That limits every choice below, especially the actuator.

Other project constraints:

- PIC18F57Q43 Curiosity Nano as the MCU. Its I/O and ADC run at its target voltage, 3.3 V as shipped ([Curiosity Nano user guide](https://www.farnell.com/datasheets/3110754.pdf), section 3.3.1).
- External 9 V barrel-jack adapter, 5 V linear regulator, fuse and power indicator.
- No daughter boards: the IMU is a bare chip on my PCB.

### Block diagram (text form)

```
9V adapter -> barrel jack -> fuse -> 9V rail -> L7805CV (with heatsink) -> 5V rail
                                                          |
         5V rail -> DRV8833 VM -> N20 gearmotor with M3 lead screw (Actuator B)
         5V rail -> Curiosity Nano VBUS -> 3.3V (VTG) -> LIS3DH IMU (SPI), slide-pot reference
         Nano PWM x2 (AIN1, AIN2), nSLEEP -> DRV8833;  DRV8833 nFAULT -> Nano
         DRV8833 sense resistor -> Nano ADC (stall detection)
         Slide pot wiper (nut position) -> Nano ADC
         LIS3DH INT1 (data ready) -> Nano interrupt input
         UART <-> 2x4 IDC ribbon to hub connector 3
```

### Connector pin table (hub connector 3, from the team block diagram)

| IDC pin | Type | Signal on my board | Direction |
|---|---|---|---|
| 1 | Digital | UART RX: Leg kinematics → linear actuator B command | Hub → my board |
| 2 | Digital | UART TX: IMU (jerk / impulse) signal and actuator status | My board → hub |
| 3 | Digital | Not used | – |
| 4 | Digital | Not used | – |
| 5 | Digital | Not used | – |
| 6 | Analog | Not used | – |
| 7 | Analog | Not used | – |
| 8 | Ground | Common ground | – |

The team message format is still to be defined in the software and message-structure assignment, so the content of the UART messages is not final. Only two signal pins are needed, which is why the I2C / serial bus in the flow diagram is implemented as a UART pair.

### Microcontroller pins (from the Block Diagram page)

| Peripheral | PIC18F57Q43 pin(s) | Connected to |
|---|---|---|
| PWM (PWM1) | RC2, RD0 | DRV8833 AIN1, AIN2 |
| Digital output | RD1 | DRV8833 nSLEEP |
| Digital input (interrupt-on-change) | RB1 | DRV8833 nFAULT |
| ADC | RA0 | DRV8833 AISEN sense resistor (0 to 0.2 V) |
| ADC | RA1 | Slide potentiometer wiper (0 to 3.3 V) |
| SPI (SPI1) | RC3 SCK, RC4 SDI, RC5 SDO, RD4 CS | LIS3DHTR |
| Digital input (INT0) | RB0 | LIS3DHTR INT1 (data ready) |
| UART (UART1) | RC7 RX, RC6 TX | Ribbon connector pins 1 and 2 |

**Labels used below:** **[Actuator B subsystem]**, **[IMU subsystem]**, **[Power subsystem]**.

---

## 2. Major component selections

### 2.1 [Actuator B subsystem] Linear actuator

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. Generic N20 6 V micro metal gearmotor with M3 lead-screw shaft (selected speed: 300 RPM)**<br><img src="https://probots.co.in/pub/media/catalog/product/cache/891768098d9d90472b8b3c7afc6da23f/n/2/n20-m3x55_1_1_1_1_1_1_1_1_1_1_1_1.jpg" width="140" alt="N20 gearmotor with M3 lead screw"><br>About $8 each (estimate from my Block Diagram page)<br>Same product family as the verified listings: [Probots 200 RPM listing](https://probots.co.in/200-rpm-6v-linear-actuator-n20-micro-metal-gear-dc-motor-with-m3-screw-lead-shaft.html) and [Probots 750 RPM listing](https://probots.co.in/750-rpm-6v-linear-actuator-n20-micro-metal-gear-dc-motor-with-m3-screw-lead-shaft.html), each ₹499 (about $6) | • Lowest cost of the options (about $8), comfortably inside a $17.50 to $20.00 per-board share<br>• Faster than the DFRobot unit at the same price class (300 RPM against 98 RPM)<br>• Rated 6 V (3 to 6 V range), so it runs from the 5 V rail through the DRV8833<br>• The lead screw turns rotation into linear motion directly, with no extra mechanism<br>• Small N20 body fits a spider leg<br>• Easy to buy in several speeds | • Slow: with a standard M3 screw of 0.5 mm pitch, 300 RPM gives about 2.5 mm/s<br>• No built-in position feedback, so it needs the slide potentiometer<br>• Sellers do not publish stall current or stall torque, so I must measure them<br>• Lead screw has backlash and is not precision<br>• I could not find a 300 RPM M3 listing with a US price, so the exact SKU must be confirmed before ordering |
| **Option 2. DFRobot Micro Metal DC Geared Motor with Lead Screw (FIT0577), 6 V, 98 RPM, M3 × 20 mm**<br><img src="https://dfimg.dfrobot.com/enshop/FIT0577/FIT0577_Main_01.jpg" width="140" alt="DFRobot FIT0577"><br>$9.00 each<br>[DFRobot product page](https://www.dfrobot.com/product-1733.html) | • Published electrical data: 24 mA no load, 420 mA stall at 6 V, which makes budgeting reliable<br>• Same 6 V N20 class as Option 1<br>• Documented supplier with a clear product page | • Only 98 RPM, so well under 1 mm/s with a standard M3 pitch<br>• The 20 mm screw limits travel<br>• Costs more than the generic motor<br>• Still has no position feedback |
| **Option 3. Pololu 298:1 Micro Metal Gearmotor LP 6 V** (plain gearmotor, no lead screw)<br><img src="https://a.pololu-files.com/picture/0J12477.1200x627.jpg?4c91aeac1932e69a79d496bb79a41e6f" width="140" alt="Pololu micro metal gearmotor"><br>$23.95 each<br>[Pololu product page](https://www.pololu.com/product/1094) | • Published specs: 40 mA no load, 0.36 A stall (extrapolated), 2.0 kg·cm stall torque<br>• High quality and well documented<br>• Strong for its size | • $23.95 for the motor alone, already above a whole board's budget share<br>• Needs a separate lead screw and coupling, which adds cost and mechanical work<br>• Only 45 RPM no load, very slow |
| **Option 4. Actuonix PQ12-P 6 V, 20 mm, 100:1, potentiometer feedback**<br><img src="https://www.robotshop.com/cdn/shop/files/pp_1781310_1200x1200.webp?v=1778285520" width="140" alt="Actuonix PQ12-P"><br>$81.76 each<br>[RobotShop product page](https://robotshop.com/products/actuonix-pq12-p-linear-actuator-20mm-1001-6v-potentiometer) · [PQ12 datasheet](https://cdn.robotshop.com/media/f/fir/rb-fir-42/pdf/actuonix-pq12-datasheet.pdf) | • Complete actuator with built-in 5 kΩ position potentiometer<br>• 40 N peak force at 6 mm/s, 550 mA stall at 6 V<br>• Well documented | • $81.76 is more than the entire $70 to $80 team budget<br>• 20% maximum duty cycle<br>• Potentiometer tolerance of ±50% needs calibration |

**Choice: Option 1, N20 6 V gearmotor with M3 lead screw [Actuator B subsystem]**

**Rationale:** The team budget is $70 to $80 for four boards, so my board has about $17.50 to $20.00 to spend in total. The generic N20 gearmotor is the lowest-cost option and about three times faster than the DFRobot unit, which costs a little more and gives nothing I need in return (its published stall data helps, but I can measure mine). The Pololu motor alone ($23.95) is more than the board's whole share, and the commercial actuator costs more than the entire team budget. Its 6 V rating works from the regulated 5 V rail through a low-drop MOSFET driver. I accept the slow speed (a leg joint demonstration does not need fast motion), the missing feedback (covered by the slide potentiometer in 2.3), and the unpublished stall data (I will measure stall current on the bench and set the driver's current limit from it). If the 300 RPM variant is hard to source, the 200 or 750 RPM listings give the same mechanical interface.

---

### 2.2 [Actuator B subsystem] Motor driver (H-bridge)

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. TI DRV8833PWPR dual H-bridge, HTSSOP-16 with PowerPAD**<br><img src="https://assets.lcsc.com/images/lcsc/900x900/20221230_Texas-Instruments-DRV8833PWPR_C50506_blank.jpg" width="140" alt="DRV8833PWPR"><br>$1.70 each at LCSC; about $2.34 at Digi-Key (price on my Block Diagram page)<br>[LCSC product page](https://www.lcsc.com/product-detail/C50506.html) · [TI datasheet](https://www.ti.com/lit/ds/symlink/drv8833.pdf) | • MOSFET bridge: only about 0.36 Ω total on-resistance at 5 V, so about 0.2 V is lost at 0.6 A and the motor sees nearly the full 5 V<br>• 2.7 to 10.8 V motor supply, 1.5 A RMS and 2 A peak per bridge, plenty of margin over a small N20<br>• Adjustable current limit with one sense resistor (0.2 V ÷ R), which also gives stall detection<br>• nFAULT output, nSLEEP pin and thermal shutdown<br>• Logic threshold of 2 V (2.5 V on nSLEEP) works with the Nano's 3.3 V outputs<br>• Very low sleep current<br>• Inexpensive | • Surface-mount only: 5.0 × 4.4 mm package with an exposed pad, which needs hot air or reflow to solder properly<br>• No built-in current sense output, so an external sense resistor and ADC reading are needed<br>• Dissipation goes through the exposed pad, so the PCB needs a copper pour under it |
| **Option 2. TI SN754410NE quadruple half-H driver, DIP-16**<br><img src="https://www.sparkfun.com/media/catalog/product/cache/6b78ac9ed927a3c2db42a3c84dab4ce5/0/0/00315-01.jpg" width="140" alt="SN754410"><br>$4.50 each<br>[SparkFun product page](https://www.sparkfun.com/h-bridge-motor-driver-1a.html) · [TI datasheet](https://www.ti.com/lit/ds/symlink/sn754410.pdf) | • Through-hole DIP, easiest to solder and replace<br>• 1 A continuous per channel, built-in clamp diodes<br>• Thermal shutdown | • Bipolar outputs lose roughly 2.6 to 3.6 V in total near 0.6 A, so on a 5 V rail a 6 V motor would see only about 1.5 to 2.5 V and barely turn<br>• No current limit or fault output<br>• About 2.6 times the cost of the DRV8833<br>• Large quiescent supply current |
| **Option 3. L293D dual H-bridge, DIP-16**<br><img src="https://www.adafruit.com/images/480x360/807-01.jpg" width="140" alt="L293D"><br>$8.95 each (price as listed)<br>[Adafruit product page](https://www.adafruit.com/product/807) · [TI datasheet](https://www.ti.com/lit/ds/symlink/l293d.pdf) | • Through-hole DIP, well documented, many examples<br>• Internal kickback diodes | • Same bipolar voltage-drop problem as Option 2, so the 6 V motor is starved on 5 V<br>• Only 600 mA continuous, with no margin against stall<br>• Most expensive option and no fault or current-limit features |

**Choice: Option 1, DRV8833PWPR [Actuator B subsystem]**

**Rationale:** The motor is fed from the 5 V rail, so the driver must drop almost nothing. Only the MOSFET-based DRV8833 does that; the bipolar DIP drivers would waste about half of the available voltage. It is also the lowest-cost option, has a built-in current limit and fault output that give me stall detection and protection (a project requirement), and works directly from the Nano's 3.3 V signals. The cost is a harder soldering job, which I accept with a stencil and hot air or reflow, and a copper pour under the exposed pad.

---

### 2.3 [Actuator B subsystem] Position sensor for the lead-screw nut

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. Bourns PTA2043-2015CPB103 slide potentiometer, 10 kΩ, 20 mm travel**<br><img src="https://mm.digikey.com/Volume0/opasdata/d220001/medias/images/2573/PTA%20SERIES%2035MM.JPG" width="140" alt="Bourns PTA2043"><br>About €1.42 each (about $1.60)<br>[Farnell product page](https://ie.farnell.com/bourns/pta2043-2015cpb103/pot-slide-10kohm-0-1w/dp/2519588) · [Digi-Key product page](https://www.digikey.com/en/products/detail/bourns-inc/PTA2043-2015CPB103/3781176) | • Gives an absolute position reading directly on one ADC pin<br>• Through-hole, simple to mount<br>• Very inexpensive<br>• Slider can be coupled to the lead-screw nut for closed-loop control<br>• Runs from the Nano's 3.3 V rail, so the signal stays within the ADC range | • Only 20 mm of travel, which limits the usable stroke<br>• ±20% resistance tolerance, so each unit needs calibration<br>• Mechanical wear and slider noise over time<br>• Needs a mechanical link between the slider and the nut |
| **Option 2. Bourns PTA4543-2015DPB103 slide potentiometer, 10 kΩ, 45 mm travel**<br><img src="https://ie.farnell.com/productimages/standard/en_GB/1688409-40.jpg" width="140" alt="Bourns PTA4543"><br>About €1.78 each<br>[Farnell product page](https://ie.farnell.com/bourns/pta4543-2015dpb103/potentiometer-slide-10k-45mm/dp/1688415) | • 45 mm of travel allows a longer stroke<br>• Same signal type and ADC interface as Option 1<br>• Still inexpensive | • Larger body is hard to fit on a small leg<br>• Travel is longer than the short N20 stroke needs, so resolution per millimetre of actuator motion is lower<br>• Same tolerance and wear limits as Option 1 |
| **Option 3 (custom). Two end-stop microswitches plus motor-current stall detection**<br>No single commercial part; cost is low but not yet quoted | • No sensing part on the leg beyond the switches<br>• Uses the sense resistor already in the design | • Only end positions are known, so there is no continuous position, which breaks closed-loop control<br>• Position between the stops would be estimated from time, which drifts<br>• Does not meet the software flow's position control |

**Choice: Option 1, PTA2043-2015CPB103 [Actuator B subsystem]**

**Rationale:** The software flow needs continuous position control, and the potentiometer is the cheapest way to get an absolute position on a single ADC pin. The 20 mm travel suits a small joint, and the slider can ride on the lead-screw nut. I accept the tolerance and wear limits and will calibrate each unit in firmware.

---

### 2.4 [IMU subsystem] Accelerometer / IMU

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. ST LIS3DHTR 3-axis accelerometer, LGA-16 (3 × 3 mm)**<br><img src="https://assets.lcsc.com/images/lcsc/900x900/20221228_STMicroelectronics-LIS3DHTR_C15134_blank.jpg" width="140" alt="LIS3DHTR"><br>$1.90 each<br>[LCSC product page](https://www.lcsc.com/product-detail/C15134.html) · [ST datasheet](https://www.st.com/resource/en/datasheet/lis3dh.pdf) | • Lowest cost of the sensors that fit the budget<br>• 1.71 to 3.6 V supply, so it runs from the Nano's 3.3 V<br>• SPI and I2C, and selectable ±2, ±4, ±8 and ±16 g ranges<br>• Data-ready interrupt lets the PIC read samples only when new data exist<br>• Gravity vector gives roll and pitch for the "orientation within range" check<br>• Fast enough to estimate jerk and impulse | • 3 × 3 mm LGA is hard to solder by hand and needs a stencil and hot air or reflow<br>• Accelerometer only, so no gyroscope and no yaw<br>• Jerk must be computed in firmware by differentiating the samples |
| **Option 2. Analog Devices ADXL345BCCZ 3-axis accelerometer, LGA-14 (3 × 5 mm)**<br><img src="https://assets.lcsc.com/images/lcsc/900x900/20221228_Analog-Devices-ADXL345BCCZ-RL7_C9667_front.jpg" width="140" alt="ADXL345"><br>$5.91 each<br>[LCSC product page](https://www.lcsc.com/product-detail/C9667.html) · [Analog Devices datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/adxl345.pdf) | • ±16 g range with SPI and I2C<br>• 2 to 3.6 V supply, works from 3.3 V<br>• Built-in shock/tap and activity detection, useful for impulse events<br>• Well documented | • About three times the price of the LIS3DH, using a large part of the board's budget share<br>• Larger 3 × 5 mm LGA is still hard to solder<br>• Still no gyroscope |
| **Option 3. TDK InvenSense MPU-6050 6-axis IMU, QFN-24 (4 × 4 mm)**<br><img src="https://assets.lcsc.com/images/lcsc/900x900/20221229_TDK-InvenSense-MPU-6050_C24112_blank.jpg" width="140" alt="MPU-6050"><br>$20.99 each<br>[LCSC product page](https://www.lcsc.com/product-detail/C24112.html) · [InvenSense datasheet](https://invensense.tdk.com/wp-content/uploads/2015/02/MPU-6000-Datasheet1.pdf) | • Adds a 3-axis gyroscope for true orientation<br>• I2C interface, 2.375 to 3.46 V supply | • $20.99 is more than a board's entire budget share<br>• I2C only<br>• 4 × 4 mm QFN with an exposed pad is the hardest to solder<br>• More capability than the jerk and impulse sensing needs |

**Choice: Option 1, LIS3DHTR [IMU subsystem]**

**Rationale:** The team assigned my board jerk and impulse sensing, which only needs acceleration, and the orientation check in the software flow can be done from the gravity vector. The LIS3DH is the cheapest option, runs from the Nano's 3.3 V rail, supports SPI with a data-ready interrupt, and fits the budget. The other two cost about three and eleven times as much for features (gyroscope, built-in tap detection) the task does not require. The soldering risk (3 × 3 mm LGA) is accepted and handled with a stencil and hot air or reflow; this is the same risk any of the three options carries.

---

### 2.5 [Power subsystem] 5 V linear voltage regulator

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. ST L7805CV 5 V regulator, TO-220**<br><img src="https://www.jameco.com/Jameco/Products/ProdImag/51262.jpg" width="140" alt="L7805CV"><br>$0.65 each<br>[Jameco product page](https://www.jameco.com/z/L7805CV-STMicroelectronics-Voltage-Regulator-5-Volt-1-5-Amp-TO-220_51262.html) · [ST datasheet](https://www.st.com/resource/en/datasheet/l78.pdf) | • Up to 1.5 A with internal current limiting and thermal shutdown<br>• Specified for 7 to 18 V input, which brackets the 9 V adapter<br>• Output capacitor is not needed for stability<br>• Very inexpensive and easy to source<br>• TO-220 accepts a heatsink | • Dropout of about 2 V at 1 A<br>• Quiescent current of about 8 mA<br>• Dissipates (9 V − 5 V) × load, about 3.5 W at my worst case, so a heatsink is required (see the [Power Budget](../05-Power-Budget/Power-Budget.md))<br>• About 50 °C/W without a heatsink |
| **Option 2. TI LM2940CT-5.0 low-dropout 5 V regulator, TO-220**<br><img src="https://mm.digikey.com/Volume0/opasdata/d220001/medias/images/3642/296%7ET03B%7ENDE%7E3.JPG" width="140" alt="LM2940CT-5.0"><br>$1.94 each<br>[Digi-Key product page](https://www.digikey.com/en/products/detail/texas-instruments/LM2940CT-5-0-NOPB/32575) · [TI datasheet](https://www.ti.com/lit/ds/symlink/lm2940.pdf) | • Dropout of about 1 V at 1 A<br>• Up to 26 V input<br>• Same TO-220 package | • About three times the cost of the L7805CV<br>• Higher quiescent current (about 15 mA)<br>• Dissipates the same heat from a 9 V input, so the lower dropout gains nothing<br>• Output capacitor requirements are stricter |

**Choice: Option 1, L7805CV [Power subsystem]**

**Rationale:** The input is a regulated 9 V adapter, so low dropout buys nothing. The L7805CV is the lowest-cost, most forgiving choice, and its built-in protection suits a board that drives a motor. The heat is the main issue: with the motor on the 5 V rail the regulator dissipates about 3.5 W in the worst case, so the TO-220 needs a heatsink of about 15 °C/W or better (the Power Budget page shows the calculation).

---

### 2.6 [Power subsystem] External 9 V power source

USB power packs are not allowed for this project, so all options are wall adapters. I already own a 9 V 1 A adapter, so it costs nothing new.

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. 9 V 1 A regulated adapter, 5.5 × 2.1 mm, center-positive (equivalent to the Adafruit unit; I already own one)**<br><img src="https://www.adafruit.com/images/480x360/63-07.jpg" width="140" alt="Adafruit 9V 1A"><br>$8.95 if bought new; $0 for me<br>[Adafruit product page](https://www.adafruit.com/product/63) | • Regulated output<br>• Polarity and barrel size are stated<br>• Already owned, so no budget spent<br>• 1 A covers my worst-case need with margin (see the Power Budget) | • Only about 107 mA of spare current after the 25% margin, so the motor current limit must be set correctly<br>• No room for added loads |
| **Option 2. Pololu 9 V 1 A regulated adapter, 5.5 × 2.1 mm, center-positive**<br><img src="https://a.pololu-files.com/picture/0J7885.1200x627.jpg?8f2dfc183198b0004956b0dbd969b6a8" width="140" alt="Pololu 9V 1A"><br>$11.11 each<br>[Pololu product page](https://www.pololu.com/product/1470) | • Regulated, polarity stated<br>• Light, small body | • Same 1 A capacity<br>• More expensive and would use new budget |
| **Option 3. SparkFun 9 V 2 A regulated adapter, 5.5 × 2.1 mm**<br><img src="https://www.sparkfun.com/media/catalog/product/cache/6b78ac9ed927a3c2db42a3c84dab4ce5/3/1/31173-Power-Supply-Feature_2.jpg" width="140" alt="SparkFun 9V 2A"><br>$10.95 each<br>[SparkFun product page](https://www.sparkfun.com/wall-adapter-power-supply-9v-2a-barrel-jack.html) | • 2 A leaves large margin<br>• Regulated | • Plug polarity is not stated on the product page<br>• Costs $10.95 of the board's $17.50 to $20.00 share<br>• More than my design needs |

**Choice: Option 1, my existing 9 V 1 A adapter [Power subsystem]**

**Rationale:** The Power Budget shows the system needs about 0.89 A including the 25% margin, which the 1 A adapter covers. It is already owned, so it uses none of the board's budget, and its polarity is known to be center-positive. This only works if the DRV8833 current limit is set to about 0.67 A rather than the 1 A in my Block Diagram page, otherwise a stalled motor could exceed the adapter's rating.

---

## 3. Cost against the team budget

| Item | Selected part | Approx. cost |
|---|---|---|
| Linear actuator | N20 6 V gearmotor with M3 lead screw | ~$8.00 |
| Motor driver | DRV8833PWPR | $2.34 |
| Position sensor | Bourns PTA2043-2015CPB103 | ~$1.60 |
| IMU | LIS3DHTR | ~$2.00 |
| 5 V regulator | L7805CV (plus a heatsink, not yet priced) | ~$0.60 |
| Fuse, barrel jack, power LED, header and passives | Per Block Diagram page | ~$3.80 |
| 9 V adapter | Already owned | $0 |
| **Total (new purchases)** | | **~$18.34 (the Block Diagram page rounds to ~$18.50)** |

The team budget is $70 to $80 for four boards, an even share of $17.50 to $20.00. This board is inside that range, and buying the N20 motor from a lower-cost supplier would bring it to about $16. The heatsink adds a small cost that is not yet included.

---

## 4. Checklist

- [x] At least 3 candidates per peripheral (the regulator list has 2 and the supply list 3)
- [x] Pictures and links present (prices as of 2026-10-19)
- [x] Multiple pros and cons per option
- [x] Rationale for each selection
- [x] Pin table included (team hub connector 3 pinout)
- [ ] Power budget: see [Power Budget](../05-Power-Budget/Power-Budget.md)
