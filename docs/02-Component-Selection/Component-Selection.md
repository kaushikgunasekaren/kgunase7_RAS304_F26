---
title: Component Selection
---

# Component Selection

**RAS 304 – Individual Datasheet (Kaushik) – Driver Nano 3: Linear Actuator B + IMU**
Project: Team 102 Modular Spider Leg (my board is the spoke on hub connector 3)

> Scope: this page covers only my individual board (Driver Nano 3). Team-level decisions are in the team repository. The selected part types are the same ones shown on my [Block Diagram](../01-Block-Diagram/Block-Diagram.md) page. Every product link below goes to a supplier that sells to customers in the United States, and prices are US dollar prices for quantity 1 checked on 2026-10-19 (they will change, and shipping and tax are not included).

## 1. Subsystem summary

Driver Nano 3 handles Linear Actuator B control and IMU sensing. It follows this loop from the team software flow diagram:

1. Receive the actuator B target position from the Main Processor (Preston's hub board).
2. Drive Linear Actuator B (PWM and direction).
3. Read the IMU orientation (SPI).
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
9V adapter -> barrel jack -> 1A fuse -> 1N5819 diode -> ~8.6V
     ~8.6V -> L7806CV -> 6V motor rail -> DRV8833 VM -> N20 gearmotor with M3 lead screw (Actuator B)
     ~8.6V -> L7805CV -> 5V logic rail -> (JP1, D3) Curiosity Nano VBUS, power LED
     Nano VTG 3.3V -> LIS3DH IMU (SPI), slide-pot reference, MCP601 op-amp
     Nano PWM x2 (AIN1, AIN2), nSLEEP -> DRV8833;  DRV8833 nFAULT -> Nano
     DRV8833 0.33 ohm sense resistor -> MCP601 (gain 11) -> Nano ADC (stall detection)
     Slide pot wiper (nut position) -> Nano ADC
     LIS3DH INT1 (data ready) -> Nano interrupt input
     Debug LEDs (RD2, RD3), push button (RD5)
     UART <-> 2x4 IDC ribbon to hub connector 3
```

The full drawing is on my [Block Diagram](../01-Block-Diagram/Block-Diagram.md) page.

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

The team message format is still to be defined in the software and message-structure assignment, so the content of the UART messages is not final. Communication with the hub is **UART** (UART1: RC7 RX, RC6 TX), not I2C. Only two signal pins plus ground are needed on the ribbon.

### Microcontroller pins (from the Block Diagram page)

| Peripheral | PIC18F57Q43 pin(s) | Connected to |
|---|---|---|
| PWM (PWM1) | RC2, RD0 | DRV8833 AIN1, AIN2 |
| Digital output | RD1 | DRV8833 nSLEEP |
| Digital input (interrupt-on-change) | RB1 | DRV8833 nFAULT |
| ADC | RA0 | MCP601 output: amplified AISEN sense voltage (0 to 2.2 V) |
| ADC | RA1 | Slide potentiometer wiper (0 to 3.3 V) |
| SPI (SPI1) | RC3 SCK, RC4 SDI, RC5 SDO, RD4 CS | LIS3DHTR |
| Digital input (INT0) | RB0 | LIS3DHTR INT1 (data ready) |
| UART (UART1) | RC7 RX, RC6 TX | Ribbon connector pins 1 and 2 |
| Digital output | RD2, RD3 | Debug LEDs |
| Digital input | RD5 | Debug push button (hardware debounced) |

**Labels used below:** **[Actuator B subsystem]**, **[IMU subsystem]**, **[Power subsystem]**.

---

## 2. Major component selections

### 2.1 [Actuator B subsystem] Linear actuator

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. DFRobot Micro Metal DC Geared Motor with Lead Screw (FIT0577), 6 V, 98 RPM, M3 × 20 mm**<br><img src="https://dfimg.dfrobot.com/enshop/FIT0577/FIT0577_Main_01.jpg" width="140" alt="DFRobot FIT0577"><br>$9.00 each ($11.76 at RobotShop)<br>[DFRobot product page](https://www.dfrobot.com/product-1733.html) · [RobotShop listing](https://www.robotshop.com/products/dfrobot-micro-metal-dc-geared-motor-w-lead-screw-6v-98rpm-m320) | • Same design as my Block Diagram choice: an N20 6 V gearmotor with the M3 lead screw built onto the shaft<br>• Ready-made part that can be ordered in the US (DFRobot, or RobotShop's US store)<br>• Published electrical data: 24 mA no load and 420 mA stall at 6 V, so the power budget is based on real numbers<br>• $9.00 fits the board's share of the team budget<br>• Rated 6 V, so it runs from the 5 V rail through the DRV8833 | • Slow: 98 RPM no load and 60 RPM under load, which is about 0.5 to 0.8 mm/s with a standard 0.5 mm M3 pitch (the page does not state the pitch)<br>• The 20 mm screw limits the stroke to under 20 mm<br>• No built-in position feedback, so it needs the slide potentiometer<br>• Shipping cost and time are not included |
| **Option 2. Aslong N20 6 V 300 RPM gearmotor (3 mm shaft) plus an M3 threaded rod and shaft coupler**<br><img src="https://www.robotshop.com/cdn/shop/files/e4d8002d_n20-gear-motor-01_1200x1200.webp?v=1720471722" width="140" alt="Aslong N20 300 RPM"><br>$6.95 for the motor; rod and coupler not priced<br>[RobotShop product page](https://robotshop.com/products/aslong-motor-dc-6v-300rpm-3mm-shaft-mini-metal-gearwheel-n20-geared-motor) | • 300 RPM, about three times the speed of Option 1 (about 2.5 mm/s with a 0.5 mm pitch)<br>• Lowest price for the motor itself<br>• Rated 6 V, all-metal gears<br>• Longer screw possible, so more stroke | • Sold as a plain gearmotor, so the lead screw, a coupler or adapter between the 3 mm shaft and the screw, and a nut must be sourced and aligned<br>• Extra cost and mechanical work that is not in the $6.95 price<br>• The page lists no stall current or torque, so the power budget would rest on an estimate<br>• More risk of misalignment and backlash |
| **Option 3. Pololu 298:1 Micro Metal Gearmotor LP 6 V** (plain gearmotor, no lead screw)<br><img src="https://a.pololu-files.com/picture/0J12477.1200x627.jpg?4c91aeac1932e69a79d496bb79a41e6f" width="140" alt="Pololu micro metal gearmotor"><br>$23.95 each<br>[Pololu product page](https://www.pololu.com/product/1094) | • Published specs: 40 mA no load, 0.36 A stall (extrapolated), 2.0 kg·cm stall torque<br>• High quality and well documented<br>• Strong for its size | • $23.95 for the motor alone, already above a whole board's budget share<br>• Needs a separate lead screw and coupling, which adds cost and mechanical work<br>• Only 45 RPM no load, very slow |
| **Option 4. Actuonix PQ12-P 6 V, 20 mm, 100:1, potentiometer feedback**<br><img src="https://www.robotshop.com/cdn/shop/files/pp_1781310_1200x1200.webp?v=1778285520" width="140" alt="Actuonix PQ12-P"><br>$81.76 each<br>[RobotShop product page](https://robotshop.com/products/actuonix-pq12-p-linear-actuator-20mm-1001-6v-potentiometer) · [PQ12 datasheet](https://cdn.robotshop.com/media/f/fir/rb-fir-42/pdf/actuonix-pq12-datasheet.pdf) | • Complete actuator with built-in 5 kΩ position potentiometer<br>• 40 N peak force at 6 mm/s, 550 mA stall at 6 V<br>• Well documented | • $81.76 is more than the entire $70 to $80 team budget<br>• 20% maximum duty cycle<br>• Potentiometer tolerance of ±50% needs calibration |

**Choice: Option 1, DFRobot FIT0577 N20 gearmotor with M3 lead screw [Actuator B subsystem]**

**Rationale:** The team budget is $70 to $80 for four boards, so my board has about $17.50 to $20.00 to spend in total, and the actuator must also be orderable in the US. Option 1 keeps the design from my Block Diagram page (an N20 6 V gearmotor driving an M3 lead screw) as one ready-made part, costs $9.00, and comes with published stall and no-load currents. Option 2 gives the original 300 RPM speed for a little less money, but it leaves me to buy and align the screw and coupler, and it has no published current data. The Pololu motor alone ($23.95) is more than the board's whole share, and the commercial actuator costs more than the entire team budget. I accept the slow speed (a joint demonstration does not need fast motion) and the missing feedback (covered by the slide potentiometer in 2.3). If the team needs more speed later, Option 2 is the fallback.

---

### 2.2 [Actuator B subsystem] Motor driver (H-bridge)

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. TI DRV8833PWPR dual H-bridge, HTSSOP-16 with PowerPAD**<br><img src="https://mm.digikey.com/Volume0/opasdata/d220001/medias/images/6222/296%7E4073225-3%7EPWP%7E16.jpg" width="140" alt="DRV8833PWPR"><br>$3.02 each (Digi-Key and Mouser)<br>[Digi-Key product page](https://www.digikey.com/en/products/detail/texas-instruments/DRV8833PWPR/2743167) · [TI datasheet](https://www.ti.com/lit/ds/symlink/drv8833.pdf) | • MOSFET bridge: only about 0.36 Ω total on-resistance at 5 V, so about 0.2 V is lost at 0.6 A and the motor sees nearly the full 5 V<br>• 2.7 to 10.8 V motor supply, 1.5 A RMS and 2 A peak per bridge, plenty of margin over a small N20<br>• Adjustable current limit with one sense resistor (0.2 V ÷ R), which also gives stall detection<br>• nFAULT output, nSLEEP pin and thermal shutdown<br>• Logic threshold of 2 V (2.5 V on nSLEEP) works with the Nano's 3.3 V outputs<br>• Very low sleep current<br>• Lowest cost of the three options | • Surface-mount only: 5.0 × 4.4 mm package with an exposed pad, which needs hot air or reflow to solder properly<br>• No built-in current sense output, so an external sense resistor and ADC reading are needed<br>• Dissipation goes through the exposed pad, so the PCB needs a copper pour under it |
| **Option 2. TI SN754410NE quadruple half-H driver, DIP-16**<br><img src="https://www.sparkfun.com/media/catalog/product/cache/6b78ac9ed927a3c2db42a3c84dab4ce5/0/0/00315-01.jpg" width="140" alt="SN754410"><br>$4.50 each<br>[SparkFun product page](https://www.sparkfun.com/h-bridge-motor-driver-1a.html) · [TI datasheet](https://www.ti.com/lit/ds/symlink/sn754410.pdf) | • Through-hole DIP, easiest to solder and replace<br>• 1 A continuous per channel, built-in clamp diodes<br>• Thermal shutdown | • Bipolar outputs lose roughly 2.6 to 3.6 V in total near 0.6 A, so on a 5 V rail a 6 V motor would see only about 1.5 to 2.5 V and barely turn<br>• No current limit or fault output<br>• About 1.5 times the cost of the DRV8833<br>• Large quiescent supply current |
| **Option 3. L293D dual H-bridge, DIP-16**<br><img src="https://www.adafruit.com/images/480x360/807-01.jpg" width="140" alt="L293D"><br>$8.95 each (price as listed)<br>[Adafruit product page](https://www.adafruit.com/product/807) · [TI datasheet](https://www.ti.com/lit/ds/symlink/l293d.pdf) | • Through-hole DIP, well documented, many examples<br>• Internal kickback diodes | • Same bipolar voltage-drop problem as Option 2, so the 6 V motor is starved on 5 V<br>• Only 600 mA continuous, with no margin against stall<br>• Most expensive option and no fault or current-limit features |

**Choice: Option 1, DRV8833PWPR [Actuator B subsystem]**

**Rationale:** The motor is fed from the 5 V rail, so the driver must drop almost nothing. Only the MOSFET-based DRV8833 does that; the bipolar DIP drivers would waste about half of the available voltage. It is also the lowest-cost option, has a built-in current limit and fault output that give me stall detection and protection (a project requirement), and works directly from the Nano's 3.3 V signals. The cost is a harder soldering job, which I accept with a stencil and hot air or reflow, and a copper pour under the exposed pad.

---

### 2.3 [Actuator B subsystem] Position sensor for the lead-screw nut

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. Bourns PTA2043-2015CPB103 slide potentiometer, 10 kΩ, 20 mm travel**<br><img src="https://mm.digikey.com/Volume0/opasdata/d220001/medias/images/2573/PTA%20SERIES%2035MM.JPG" width="140" alt="Bourns PTA2043"><br>$1.84 each (Newark and Avnet)<br>[Digi-Key product page](https://www.digikey.com/en/products/detail/bourns-inc/PTA2043-2015CPB103/3781176) | • Gives an absolute position reading directly on one ADC pin<br>• Through-hole, simple to mount<br>• Very inexpensive<br>• Slider can be coupled to the lead-screw nut for closed-loop control<br>• Runs from the Nano's 3.3 V rail, so the signal stays within the ADC range | • Only 20 mm of travel, which limits the usable stroke<br>• ±20% resistance tolerance, so each unit needs calibration<br>• Mechanical wear and slider noise over time<br>• Needs a mechanical link between the slider and the nut |
| **Option 2. Bourns PTA4543-2015DPB103 slide potentiometer, 10 kΩ, 45 mm travel**<br><img src="https://ie.farnell.com/productimages/standard/en_GB/1688409-40.jpg" width="140" alt="Bourns PTA4543"><br>$2.21 each (Avnet), $2.35 (Newark)<br>[Digi-Key product page](https://www.digikey.com/en/products/detail/bourns-inc/PTA4543-2015DPB103/3534245) | • 45 mm of travel allows a longer stroke<br>• Same signal type and ADC interface as Option 1<br>• Still inexpensive | • Larger body is hard to fit on a small leg<br>• Travel is longer than the short N20 stroke needs, so resolution per millimetre of actuator motion is lower<br>• Same tolerance and wear limits as Option 1 |
| **Option 3 (custom). Two end-stop microswitches plus motor-current stall detection**<br>No single commercial part; cost is low but not yet quoted | • No sensing part on the leg beyond the switches<br>• Uses the sense resistor already in the design | • Only end positions are known, so there is no continuous position, which breaks closed-loop control<br>• Position between the stops would be estimated from time, which drifts<br>• Does not meet the software flow's position control |

**Choice: Option 1, PTA2043-2015CPB103 [Actuator B subsystem]**

**Rationale:** The software flow needs continuous position control, and the potentiometer is the lowest-cost way to get an absolute position on a single ADC pin. The 20 mm travel suits a small joint, and the slider can ride on the lead-screw nut. I accept the tolerance and wear limits and will calibrate each unit in firmware.

---

### 2.4 [IMU subsystem] Accelerometer / IMU

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. ST LIS3DHTR 3-axis accelerometer, LGA-16 (3 × 3 mm)**<br><img src="https://mm.digikey.com/Volume0/opasdata/d220001/medias/images/332/497%3B-7983231%3B-%3B-16.JPG" width="140" alt="LIS3DHTR"><br>$1.94 each (Newark and Avnet)<br>[Digi-Key product page](https://www.digikey.com/en/products/detail/stmicroelectronics/LIS3DHTR/2334338) · [ST datasheet](https://www.st.com/resource/en/datasheet/lis3dh.pdf) | • Lowest cost of the sensors that fit the budget<br>• 1.71 to 3.6 V supply, so it runs from the Nano's 3.3 V<br>• SPI and I2C, and selectable ±2, ±4, ±8 and ±16 g ranges<br>• Data-ready interrupt lets the PIC read samples only when new data exist<br>• Gravity vector gives roll and pitch for the "orientation within range" check<br>• Fast enough to estimate jerk and impulse | • 3 × 3 mm LGA is hard to solder by hand and needs a stencil and hot air or reflow<br>• Accelerometer only, so no gyroscope and no yaw<br>• Jerk must be computed in firmware by differentiating the samples |
| **Option 2. Analog Devices ADXL345BCCZ 3-axis accelerometer, LGA-14 (3 × 5 mm)**<br><img src="https://assets.lcsc.com/images/lcsc/900x900/20221228_Analog-Devices-ADXL345BCCZ-RL7_C9667_front.jpg" width="140" alt="ADXL345"><br>$8.66 each at Newark ($7.05 at Arrow, $10.36 at Mouser)<br>[Digi-Key product page](https://www.digikey.com/en/products/detail/analog-devices-inc/ADXL345BCCZ-RL7/ADXL345BCCZ-RL7CT-ND/2038984) · [Analog Devices datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/adxl345.pdf) | • ±16 g range with SPI and I2C<br>• 2 to 3.6 V supply, works from 3.3 V<br>• Built-in shock/tap and activity detection, useful for impulse events<br>• Well documented | • About four times the price of the LIS3DH, using a large part of the board's budget share<br>• Larger 3 × 5 mm LGA is still hard to solder<br>• Still no gyroscope |
| **Option 3. TDK InvenSense MPU-6050 6-axis IMU, QFN-24 (4 × 4 mm)**<br><img src="https://assets.lcsc.com/images/lcsc/900x900/20221229_TDK-InvenSense-MPU-6050_C24112_blank.jpg" width="140" alt="MPU-6050"><br>US price not confirmed (I found $5.78 at one reseller and $20.99 at LCSC)<br>[Digi-Key product page](https://www.digikey.com/en/products/detail/tdk-invensense/MPU-6050/4038009) · [InvenSense datasheet](https://invensense.tdk.com/wp-content/uploads/2015/02/MPU-6000-Datasheet1.pdf) | • Adds a 3-axis gyroscope for true orientation<br>• I2C interface, 2.375 to 3.46 V supply | • Even at the lowest price I found it is about three times the cost of the LIS3DH, and the price varies widely between sellers<br>• I2C only<br>• 4 × 4 mm QFN with an exposed pad is the hardest to solder<br>• More capability than the jerk and impulse sensing needs |

**Choice: Option 1, LIS3DHTR [IMU subsystem]**

**Rationale:** The team assigned my board jerk and impulse sensing, which only needs acceleration, and the orientation check in the software flow can be done from the gravity vector. The LIS3DH is the lowest-cost option, runs from the Nano's 3.3 V rail, supports SPI with a data-ready interrupt, and fits the budget. The other two cost at least about three to four times as much for features (gyroscope, built-in tap detection) the task does not require. The soldering risk (3 × 3 mm LGA) is accepted and handled with a stencil and hot air or reflow; this is the same risk any of the three options carries.

---

### 2.5 [Power subsystem] 5 V linear voltage regulator

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. ST L7805CV 5 V regulator, TO-220**<br><img src="https://www.jameco.com/Jameco/Products/ProdImag/51262.jpg" width="140" alt="L7805CV"><br>$0.65 each<br>[Jameco product page](https://www.jameco.com/z/L7805CV-STMicroelectronics-Voltage-Regulator-5-Volt-1-5-Amp-TO-220_51262.html) · [ST datasheet](https://www.st.com/resource/en/datasheet/l78.pdf) | • Up to 1.5 A with internal current limiting and thermal shutdown<br>• Specified for 7 to 18 V input, which brackets the 9 V adapter<br>• Output capacitor is not needed for stability<br>• Very inexpensive and easy to source<br>• TO-220 accepts a heatsink | • Dropout of about 2 V at 1 A<br>• Quiescent current of about 8 mA<br>• Dissipates (9 V − 5 V) × load; this was 2.6 W when the motor shared the 5 V rail, and is now about 0.5 W (see the [Power Budget](../05-Power-Budget/Power-Budget.md))<br>• About 50 °C/W without a heatsink |
| **Option 2. TI LM2940CT-5.0 low-dropout 5 V regulator, TO-220**<br><img src="https://mm.digikey.com/Volume0/opasdata/d220001/medias/images/3642/296%7ET03B%7ENDE%7E3.JPG" width="140" alt="LM2940CT-5.0"><br>$1.94 each<br>[Digi-Key product page](https://www.digikey.com/en/products/detail/texas-instruments/LM2940CT-5-0-NOPB/32575) · [TI datasheet](https://www.ti.com/lit/ds/symlink/lm2940.pdf) | • Dropout of about 1 V at 1 A<br>• Up to 26 V input<br>• Same TO-220 package | • About three times the cost of the L7805CV<br>• Higher quiescent current (about 15 mA)<br>• Dissipates the same heat from a 9 V input, so the lower dropout gains nothing<br>• Output capacitor requirements are stricter |

**Choice: Option 1, L7805CV [Power subsystem]**

**Rationale:** The input is a regulated 9 V adapter, so low dropout buys nothing. The L7805CV is the lowest-cost, most forgiving choice, and its built-in protection suits a board that drives a motor. Heat was the main issue while the motor shared the 5 V rail (about 2.6 W). After the motor moved to its own L7806CV (section 2.7), the L7805CV carries only about 0.13 A and needs no heatsink.

---

### 2.6 [Power subsystem] External 9 V power source

USB power packs are not allowed for this project, so all options are wall adapters. I already own a 9 V 1 A adapter, so it costs nothing new.

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. 9 V 1 A regulated adapter, 5.5 × 2.1 mm, center-positive (equivalent to the Adafruit unit; I already own one)**<br><img src="https://www.adafruit.com/images/480x360/63-07.jpg" width="140" alt="Adafruit 9V 1A"><br>$8.95 if bought new; $0 for me<br>[Adafruit product page](https://www.adafruit.com/product/63) | • Regulated output<br>• Polarity and barrel size are stated<br>• Already owned, so no budget spent<br>• 1 A covers my worst-case need with margin (see the Power Budget) | • About 326 mA of spare current after the 25% margin in normal operation, and about 88 mA with the motor at its 0.61 A current limit (see the Power Budget)<br>• No room for large added loads |
| **Option 2. Pololu 9 V 1 A regulated adapter, 5.5 × 2.1 mm, center-positive**<br><img src="https://a.pololu-files.com/picture/0J7885.1200x627.jpg?8f2dfc183198b0004956b0dbd969b6a8" width="140" alt="Pololu 9V 1A"><br>$11.11 each<br>[Pololu product page](https://www.pololu.com/product/1470) | • Regulated, polarity stated<br>• Light, small body | • Same 1 A capacity<br>• More expensive and would use new budget |
| **Option 3. SparkFun 9 V 2 A regulated adapter, 5.5 × 2.1 mm**<br><img src="https://www.sparkfun.com/media/catalog/product/cache/6b78ac9ed927a3c2db42a3c84dab4ce5/3/1/31173-Power-Supply-Feature_2.jpg" width="140" alt="SparkFun 9V 2A"><br>$10.95 each<br>[SparkFun product page](https://www.sparkfun.com/wall-adapter-power-supply-9v-2a-barrel-jack.html) | • 2 A leaves large margin<br>• Regulated | • Plug polarity is not stated on the product page<br>• Costs $10.95 of the board's $17.50 to $20.00 share<br>• More than my design needs |

**Choice: Option 1, my existing 9 V 1 A adapter [Power subsystem]**

**Rationale:** The Power Budget shows the system needs about 0.67 A including the 25% margin and the regulators' own current, which the 1 A adapter covers. It is already owned, so it uses none of the board's budget, and its polarity is known to be center-positive. In a motor fault the limit matters: the DRV8833 current limit is set to about 0.61 A with a 0.33 Ω sense resistor, so even a fault stays within the adapter's rating.

---

### 2.7 [Power subsystem] Motor supply rail (added after the power budget)

My first design ran the 6 V motor from the 5 V rail. The power budget showed two problems: the L7805CV would dissipate about 2.6 W and need a heatsink, and the motor would run below its rated voltage. I compared three ways to supply the motor:

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. Second linear regulator, ST L7806CV (6 V, TO-220)**<br>[ST L78 datasheet](https://www.st.com/resource/en/datasheet/l78.pdf) | • Motor runs at its rated 6 V<br>• Motor current and stall spikes stay off the microcontroller's 5 V rail<br>• Same family, footprint and protection (current limit, thermal shutdown) as the L7805CV<br>• Inexpensive | • Needs at least 8 V in; about 8.6 V is available from the 9 V adapter after the diode<br>• About 1.4 W of heat at stall from 9 V, more from a 12 V adapter |
| **Option 2. Keep the motor on the L7805CV 5 V rail** (original design) | • No extra part | • L7805CV dissipates about 2.6 W and needs a heatsink<br>• Motor runs under-voltage<br>• Motor noise and stall dips reach the Nano's supply |
| **Option 3. Run the DRV8833 from the raw 9 V rail and cap the PWM duty near 66%** | • No regulator heat at all | • Applies 9 V peaks to a 6 V motor<br>• A firmware bug or reset at full duty could overdrive the motor<br>• Supply varies with the adapter used |

**Choice: Option 1, L7806CV [Power subsystem]**

**Rationale:** It is the only option that gives the motor its rated voltage while protecting the microcontroller's rail, and it adds well under a dollar. It also moved most of the heat off the L7805CV, which now needs no heatsink. See the [Power Budget](../05-Power-Budget/Power-Budget.md).

---

### 2.8 [Actuator B subsystem] Current-sense signal conditioning (added in the schematic)

The 0.33 Ω sense resistor produces only 0 to 0.2 V at the current limit. Read directly, that uses about 6% of the 3.3 V ADC range, which is too coarse for a reliable stall threshold.

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. Microchip MCP601-I/P op-amp, non-inverting, gain 11 (PDIP-8)**<br>[Microchip MCP601 datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/21314g.pdf) | • Single-supply, rail-to-rail output, runs from the Nano's 3.3 V<br>• 0 to 0.2 V becomes 0 to 2.2 V, using about two-thirds of the ADC range<br>• Through-hole, easy to solder<br>• Very low supply current (about 0.3 mA) | • Adds an IC and a few resistors<br>• Small offset error needs a zero-current calibration |
| **Option 2. Read the sense resistor directly on the ADC** | • No extra parts | • Only about 250 ADC counts (of 4096) across the full current range<br>• Noisy, unreliable stall threshold |
| **Option 3. Dedicated current-sense amplifier IC (for example TI INA180)** | • Fixed, accurate gain | • Surface-mount only<br>• Higher cost than a general op-amp<br>• Low-side sensing here does not need its common-mode range |

**Choice: Option 1, MCP601-I/P [Actuator B subsystem]**

**Rationale:** It gives the PIC a usable signal for stall detection (Requirement 6.1) with one through-hole part that runs from the existing 3.3 V rail. A 159 Hz RC filter before it and a 1.6 kHz anti-alias filter after it remove PWM ripple.

---

## 3. MCC configuration (PIC18F57Q43)

The microcontroller's peripherals are set up in MPLAB Code Configurator (MCC). Every subsystem and pin used on the board is listed below.

| MCC module | Setting | Pins |
|---|---|---|
| Clock (CLKCTRL) | HFINTOSC, 64 MHz | – |
| PWM1 (16-bit PWM) | About 20 kHz (above hearing range, within DRV8833 limit), two outputs | RC2 → AIN1, RD0 → AIN2 |
| ADCC | 12-bit, VDD (3.3 V) reference, right-justified | RA0 (ANA0, motor current), RA1 (ANA1, position) |
| SPI1 (host mode) | Mode 3 (CPOL = 1, CPHA = 1), 1 MHz | RC3 SCK, RC4 SDI, RC5 SDO; RD4 GPIO as chip select |
| UART1 | 115200 baud, 8N1, receive interrupt on | RC7 RX, RC6 TX |
| INT0 (external interrupt) | Rising edge | RB0 (LIS3DH INT1 data ready) |
| Interrupt-on-change | Falling edge | RB1 (DRV8833 nFAULT) |
| GPIO outputs | Push-pull, start low | RD1 (nSLEEP), RD2, RD3 (debug LEDs), RD4 (IMU CS, starts high) |
| GPIO input | Digital input | RD5 (debug button) |
| GPIO (spare) | Unused pins left as inputs, broken out on J5 and J6 | RB2–RB5, RA2–RA7, RE0–RE2, RF2–RF7, RD6, RD7, RC0, RC1 |

---

## 4. Cost against the team budget

| Item | Selected part | Approx. cost |
|---|---|---|
| Linear actuator | DFRobot FIT0577 N20 gearmotor with M3 lead screw | $9.00 |
| Motor driver | DRV8833PWPR | $3.02 |
| Position sensor | Bourns PTA2043-2015CPB103 | $1.84 |
| IMU | LIS3DHTR | $1.94 |
| 6 V regulator (motor) | L7806CV | ~$0.65 |
| 5 V regulator (logic) | L7805CV (no heatsink needed now) | $0.65 |
| Current-sense amplifier | MCP601-I/P | ~$0.50 |
| Reverse-polarity diode | 1N5819RLG | ~$0.10 |
| Fuse, barrel jack, power LED, header and passives | Estimates from my Block Diagram page | ~$3.80 |
| 9 V adapter | Already owned | $0 |
| **Total (new purchases, before shipping)** | | **~$21.50** |

The team budget is $70 to $80 for four boards, an even share of $17.50 to $20.00. At about $21.50 this board is slightly above that range, mainly because of the 6 V regulator and op-amp added for protection, so I should check the team total with my teammates. The fuse, jack, LED and passive costs are estimates and may be lower. Ordering the Option 2 motor ($6.95) would save about $2, but it needs a threaded rod and coupler.

---

## 5. How the selected components meet the product requirements

I chose each part by first ruling out options that broke a hard constraint (the PIC18F57Q43, a 9 V linear-regulated supply, no daughter boards, US-orderable parts), then ranking the rest by cost against the board's share of the team budget, and finally by how well they serve the team requirements below.

| Team requirement | Parts that meet it |
|---|---|
| **2.2** Standard communication protocols | UART to the hub; SPI to the LIS3DH |
| **4.4 / 5.2** Off-the-shelf, replaceable parts | Catalog motor (FIT0577) and slide pot on headers; through-hole regulators, op-amp and connectors |
| **6.1** Actuator current limiting or stall detection | DRV8833 current limit (0.61 A) and nFAULT; MCP601 amplifies the sense voltage for an ADC stall check |
| **6.3** Over-voltage and over-current protection | 1 A fuse, 1N5819 reverse-polarity diode, L7806CV/L7805CV current limit and thermal shutdown |
| Team budget ($70–80 for four boards) | Lowest-cost option chosen wherever it met the requirements (actuator, driver, IMU, regulators); adapter already owned |
| Jerk / impulse sensing (team-assigned task) | LIS3DH with data-ready interrupt, fast enough to differentiate acceleration into jerk |

## 6. Checklist

- [x] At least 3 candidates per peripheral (the regulator list has 2 and the supply list 3)
- [x] Pictures and links present (prices as of 2026-10-19)
- [x] Multiple pros and cons per option
- [x] Rationale for each selection
- [x] Pin table included (team hub connector 3 pinout)
- [x] Power budget: see [Power Budget](../05-Power-Budget/Power-Budget.md)
- [x] MCC configuration: see section 3
