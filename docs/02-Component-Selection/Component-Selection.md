---
title: Component Selection
---

# Component Selection

**RAS 304 – Individual Datasheet (Kaushik) – Driver Nano 2: Linear Actuator A + Pressure Sensor**
Project: Single Robotic Spider Leg

> Scope: this page covers only my individual board (Driver Nano 2). Team-level decisions live in the team repository. Prices were checked on 2026-10-19 and will change.

## 1. Subsystem summary

Driver Nano 2 handles Linear Actuator A control and pressure sensing, and follows this loop from the team software flow diagram:

1. Receive the actuator A target position from the Main Processor over I2C / serial.
2. Drive Linear Actuator A (PWM and direction).
3. Read the pressure sensor (analog or digital).
4. Check whether contact is detected or the force threshold is reached. If yes, hold or adjust the actuator (position control). If no, continue the motion.
5. Send actuator A status and pressure data back to the Main Processor over I2C / serial.
6. Loop, waiting for the next command.

When the Main Processor sends a stop command, Driver Nano 2 disables the driver and reports a safe status. The Main Processor does the kinematics; Driver Nano 2 only runs local control and returns feedback continuously.

Project constraints that drive the selections:

- Through-hole parts only (no daughter boards), PIC18F57Q43 Curiosity Nano as the MCU.
- External 9 V barrel-jack supply, on-board 5 V linear regulator, fuse and power indicator.
- The Curiosity Nano runs its I/O and ADC at its target voltage, which is 3.3 V as shipped ([Curiosity Nano user guide](https://www.farnell.com/datasheets/3110754.pdf), section 3.3.1). Every analog signal into the MCU must stay at or below 3.3 V.
- 2x4 IDC bus connector to the Main Processor Nano, carrying I2C / serial per the software flow diagram.

Assumptions to confirm with the team (they change the actuator and driver choices if they move):

- Actuator A needs about 20 mm of stroke and no more than about 40 N at the foot.
- The foot sensor is used for contact and threshold detection (per the software flow), not for calibrated force measurement.

### Block diagram (text form)

```
9V barrel jack -> fuse -> 9V rail --+--------------------> SN754410 VCC2 -> Linear Actuator A (PQ12-P)
                                    |
                                    +-> L7805CV -> 5V rail -> Curiosity Nano (VBUS) -> 3.3V (VTG)
                                                                  |                       |
                         PWM + 2 direction pins ------------------+                       +-> actuator pot reference
                         SN754410 logic supply (5V)                                       +-> MCP6002 supply
                         2 ADC inputs <- actuator pot wiper, FSR 402 via MCP6002 (analog read)
                         1 digital input <- MCP6002 spare amplifier as comparator (contact flag, digital read)
                         I2C / serial <-> 2x4 IDC bus to Main Processor
```

### Pin table (2x4 IDC team bus) – DRAFT, must match the team block diagram

| IDC pin | Type | Signal on my board | Direction |
|---|---|---|---|
| 1 | Digital I/O | I2C SDA: target position commands in, actuator status and pressure data out | Bidirectional |
| 2 | Digital I/O | I2C SCL | In (clock from Main Processor) |
| 3 | Digital I/O | Serial (UART) RX, alternate to I2C if the team chooses serial | In |
| 4 | Digital I/O | Serial (UART) TX, alternate to I2C if the team chooses serial | Out |
| 5 | Digital I/O | Not used (the stop command arrives as a bus message) | – |
| 6 | Analog I/O | Not used (reserved for pressure signal pass-through) | – |
| 7 | Analog I/O | Not used (reserved for actuator position pass-through) | – |
| 8 | Ground | Common ground | – |

Only one of I2C or serial is used at a time, as the team decides. If I2C is used, the pull-up resistors go on one board only, to be agreed with the team.

MCU resources needed on my board (pin mapping is done in the schematic assignment): 1 PWM output, 2 digital outputs (direction), 2 ADC inputs (actuator position, pressure), 1 digital input (contact flag), and the I2C or UART pins for the bus.

**Labels used below:** **[Actuator A subsystem]**, **[Pressure sensor subsystem]**, **[Power subsystem]**.

---

## 2. Major component selections

### 2.1 [Actuator A subsystem] Linear actuator with position feedback

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. Actuonix PQ12-P, 20 mm, 100:1, 6 V, potentiometer feedback**<br><img src="https://www.robotshop.com/cdn/shop/files/pp_1781310_1200x1200.webp?v=1778285520" width="140" alt="Actuonix PQ12-P 6V"><br>$81.76 each<br>[RobotShop product page](https://robotshop.com/products/actuonix-pq12-p-linear-actuator-20mm-1001-6v-potentiometer) · [PQ12 datasheet](https://cdn.robotshop.com/media/f/fir/rb-fir-42/pdf/actuonix-pq12-datasheet.pdf) | • Rated 6 V, which matches what the motor sees from a 9 V rail after the bipolar driver's saturation drop (see choice rationale)<br>• Built-in 5 kΩ position potentiometer gives closed-loop feedback with one ADC pin and no extra sensor<br>• Peak power point of 40 N at 6 mm/s, 10 mm/s no load, enough for a leg joint assumption of about 40 N<br>• Stall current of only 550 mA at 6 V keeps the supply and driver small<br>• Compact 20 mm stroke body, easy to mount on a printed leg | • 20% maximum duty cycle in the datasheet, so firmware must limit continuous driving<br>• Potentiometer is 5 kΩ ±50%, so each unit needs calibration<br>• Datasheet gives no input voltage tolerance, so firmware must cap PWM duty<br>• Expensive for a student project<br>• Only 20 mm of stroke limits leg range |
| **Option 2. Actuonix PQ12-P, 20 mm, 100:1, 12 V, potentiometer feedback**<br><img src="https://www.robotshop.com/cdn/shop/files/pp_1698337_1200x1200.webp?v=1780954972" width="140" alt="Actuonix PQ12-P 12V"><br>$81.76 each<br>[RobotShop product page](https://www.robotshop.com/products/actuonix-pq12-p-linear-actuator-20mm-1001-12v-potentiometer) · [PQ12 datasheet](https://cdn.robotshop.com/media/f/fir/rb-fir-42/pdf/actuonix-pq12-datasheet.pdf) | • Same body, feedback pot and mounting as Option 1<br>• Low stall current of 210 mA at 12 V, easy on the supply<br>• Peak power point of 40 N at 6 mm/s at its rated 12 V | • Needs 12 V, but the project supply is 9 V and the driver drops about 2.5 to 3.5 V, so the motor would see only about 6 V, half its rating, and lose speed and force<br>• Adding a boost converter would break the single 9 V and 5 V rail plan<br>• Same 20% duty cycle and pot tolerance limits as Option 1<br>• Same price as the 6 V version with no benefit at 9 V |
| **Option 3. Actuonix L12-P, 50 mm, 100:1, 12 V, analog position feedback**<br><img src="https://www.robotshop.com/cdn/shop/files/pp_1698299_1200x1200.webp?v=1778261967" width="140" alt="Actuonix L12 50mm"><br>$100.63 each<br>[RobotShop product page (datasheet linked from the listing)](https://www.robotshop.com/products/actuonix-l12-linear-actuator-50mm-1001-12v-analog) | • 50 mm of stroke gives more leg travel<br>• Fast: 23 mm/s no load<br>• Low 200 mA stall current at 12 V<br>• Analog position output suits the ADC | • 12 V rating has the same under-voltage problem as Option 2<br>• Peak power point is only 23 N at 6 mm/s, lower than the PQ12<br>• Largest and most expensive option, harder to fit on a spider leg<br>• Longer body adds mass at the joint |
| **Option 4 (custom). DC gearmotor + lead screw + linear potentiometer, built in-house**<br>Not a commercial part, no single product link or unit price | • Fully tunable stroke, speed and force<br>• Could run directly at 6 V to 9 V<br>• Parts are individually replaceable | • Needs a custom mechanical design (screw, nut, guides, coupling), which is a large schedule risk this semester<br>• Needs its own position sensing and end stops<br>• Backlash and alignment problems likely<br>• Total cost and size are unknown until designed |

**Choice: Option 1, Actuonix PQ12-P 6 V, 100:1 [Actuator A subsystem]**

**Rationale:** The bipolar H-bridge I chose (section 2.2) drops roughly 2.6 to 3.6 V in total near 0.6 A (high-side 1.4 to 1.8 V plus low-side 1.2 to 1.8 V, datasheet ranges for this driver family), so a 9 V rail delivers about 5.4 to 6.4 V to the motor. That matches the 6 V rating, so the actuator reaches its rated 40 N and 10 mm/s peak-power point. The 12 V versions would run at about half their rated voltage. The built-in potentiometer lets Driver Nano 2 report position back to the Main Processor with a single ADC pin. I accept the 20% duty cycle and the pot tolerance as limits, and handle them with firmware duty-cycle limiting and a per-unit calibration. The 6 V actuator on the 9 V rail needs a bench check of the motor-terminal voltage before full-speed testing.

---

### 2.2 [Actuator A subsystem] Motor driver (H-bridge)

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. TI SN754410NE quadruple half-H driver, DIP-16**<br><img src="https://www.sparkfun.com/media/catalog/product/cache/6b78ac9ed927a3c2db42a3c84dab4ce5/0/0/00315-01.jpg" width="140" alt="SN754410"><br>$4.50 each<br>[SparkFun product page](https://www.sparkfun.com/h-bridge-motor-driver-1a.html) · [TI datasheet](https://www.ti.com/lit/ds/symlink/sn754410.pdf) | • 1 A continuous per channel (2 A peak), about 1.7 times the 550 to 600 mA actuator stall current<br>• Built-in clamp diodes, so no external flyback diodes<br>• Logic high threshold of 2 V minimum works with the Nano's 3.3 V outputs<br>• 4.5 to 36 V motor supply covers the 9 V rail<br>• Thermal shutdown built in<br>• Through-hole DIP-16, pin-compatible with the L293D as a fallback | • Bipolar outputs drop about 2.6 to 3.6 V total, so efficiency is poor and the driver dissipates roughly 1.5 to 2 W at stall<br>• Needs the four ground pins soldered to a copper pour for heat sinking<br>• Quiescent supply current is high for a logic IC (tens of mA)<br>• No built-in current sensing |
| **Option 2. L293D dual H-bridge, DIP-16**<br><img src="https://www.adafruit.com/images/480x360/807-01.jpg" width="140" alt="L293D"><br>$8.95 each (price as listed)<br>[Adafruit product page](https://www.adafruit.com/product/807) · [TI datasheet](https://www.ti.com/lit/ds/symlink/l293d.pdf) | • Internal kickback diodes<br>• 4.5 to 36 V motor supply<br>• Well documented, many examples<br>• Same DIP-16 footprint as Option 1 | • Only 600 mA continuous per channel, which is no margin against a 550 to 600 mA stall<br>• 1.2 A peak limit is only for 100 µs<br>• Logic high threshold of 2.3 V is closer to the 3.3 V output level<br>• More expensive than the SN754410 on this listing |
| **Option 3. ST L298N dual full bridge, Multiwatt-15**<br><img src="https://mexico.newark.com/productimages/standard/en_GB/403295-40.jpg" width="140" alt="L298N"><br>$14.42 each<br>[Newark product page](https://mexico.newark.com/en-MX/stmicroelectronics/l298n/mtr-driver-40-to-150degc-multiwatt/dp/10WX1394) · [ST product page and datasheet](https://www.st.com/en/motor-drivers/l298.html) | • 2 A continuous per channel, large margin<br>• Sense pins allow a shunt resistor for current monitoring<br>• Handles up to 46 V | • Needs four external fast diodes for the inductive load<br>• Total saturation drop up to 3.2 V at 1 A, which is large for a 9 V rail<br>• High quiescent current on both supplies (tens of mA)<br>• Bulky vertical Multiwatt package and the most expensive option<br>• Overkill for a 550 mA actuator |
| **Option 4 (custom). Discrete MOSFET H-bridge (logic-level N-FETs and P-FETs with gate drive)**<br>Not a single commercial part, no unit price | • Very low voltage drop, so the motor sees nearly the full 9 V and less heat is dissipated<br>• Scales to higher current later | • Needs shoot-through protection and level-shifted gate drive for the high side<br>• Many more parts and board area<br>• Higher design and debug risk on a first board<br>• With almost no drop, a 6 V motor would see nearly the full 9 V, so the PWM duty would have to be capped tightly to avoid overvoltage |

**Choice: Option 1, SN754410NE [Actuator A subsystem]**

**Rationale:** It is the only through-hole driver here that gives real current margin (1 A continuous against about 0.6 A stall) without external diodes or heavy packaging. Its saturation drop is the reason the 6 V actuator works from a 9 V rail. The 2 V logic threshold is compatible with the Nano's 3.3 V I/O, and it is pin-compatible with the L293D if I need a different supplier. The drawbacks (power dissipation and quiescent current) are handled by a copper pour on the ground pins, firmware duty-cycle limits, and the budget in the [Power Budget](../05-Power-Budget/Power-Budget.md) page.

---

### 2.3 [Pressure sensor subsystem] Pressure / force sensor

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. Interlink FSR 402 force-sensing resistor**<br><img src="https://a.pololu-files.com/picture/0J2691.1200x627.jpg?7ea1acb355f5fbcd00b05ddd32939498" width="140" alt="FSR 402"><br>$13.49 each<br>[Pololu product page](https://www.pololu.com/product/1696) · [Interlink manufacturer page](https://www.interlinkelectronics.com/products/fsr-x-402-short) | • Responds from about 0.2 N to 20 N, which suits foot contact detection<br>• 0.58 in diameter active area is large enough to catch a foot strike<br>• Thin and flexible, easy to mount on the foot<br>• Simple resistive output, so a divider and one op-amp stage is enough<br>• Lower total cost than the load cell once the load cell's instrumentation amplifier is counted, and about 40% less than the FlexiForce | • Resistance falls from over 1 MΩ unloaded to hundreds of ohms under heavy pressure, so the front end must handle a wide range<br>• Limited absolute accuracy and drift, so it suits thresholds better than calibrated force<br>• Fragile tail and connector area, needs strain relief |
| **Option 2. Tekscan FlexiForce A201-100 (100 lb)**<br><img src="https://www.sparkfun.com/media/catalog/product/cache/6b78ac9ed927a3c2db42a3c84dab4ce5/0/8/08685-03-L.jpg" width="140" alt="FlexiForce A201"><br>$22.95 each<br>[SparkFun product page](https://www.sparkfun.com/flexiforce-pressure-sensor-100lbs.html) · [Datasheet](https://cdn.sparkfun.com/datasheets/Sensors/ForceFlex/FLX-A201-A.pdf) | • Better repeatability than a typical FSR for the same circuit<br>• 0 to 100 lb range survives heavy foot loads<br>• Published datasheet with a recommended op-amp circuit | • 100 lb range is far above the expected foot load, so light contact gives a small signal<br>• About 70% more expensive than the FSR 402<br>• Small sensing area at the tip can miss off-center contact<br>• Still needs op-amp conditioning |
| **Option 3. SparkFun TAL220 10 kg bar load cell**<br><img src="https://www.sparkfun.com/media/catalog/product/cache/6b78ac9ed927a3c2db42a3c84dab4ce5/1/3/13329-01.jpg" width="140" alt="TAL220 load cell"><br>$12.95 each<br>[SparkFun product page](https://www.sparkfun.com/load-cell-10kg-straight-bar-tal220.html) · [Datasheet](https://cdn.sparkfun.com/datasheets/Sensors/ForceFlex/TAL220M4M5Update.pdf) | • Most accurate and linear of the three, suitable for real force measurement<br>• 10 kg capacity with strain-gauge bridge, so stable over time<br>• Inexpensive for a bridge sensor of this accuracy | • Millivolt-level bridge output needs a custom instrumentation amplifier with several op-amps or a dedicated IC, which adds board area and risk<br>• 80 mm bar needs a proper mechanical mount and load path, awkward on a small leg foot<br>• Heavy compared with a film sensor<br>• Overscoped for contact detection |

**Choice: Option 1, Interlink FSR 402 [Pressure sensor subsystem]**

**Rationale:** The software flow only asks whether contact is detected or a threshold is reached, so absolute accuracy matters less than sensitivity at low force and a simple front end. The FSR 402's 0.2 N to 20 N range matches a foot contact event, it is flat and light enough for a leg, and it has the lowest total cost once the load cell's extra amplifier circuitry is counted. The load cell would be more accurate but needs a custom instrumentation amplifier and a mount the leg does not have room for. The FlexiForce's 100 lb range is too coarse for light contact. I accept the FSR's drift and nonlinearity because the firmware compares against a threshold and can recalibrate the baseline at startup.

---

### 2.4 [Pressure sensor subsystem] Signal-conditioning op-amp

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. Microchip MCP6002-I/P dual op-amp, DIP-8**<br><img src="https://ce8dc832c.cloudimg.io/v7/_cdn_/E9/D2/B0/00/0/732574_1.jpg?width=640&height=480&wat=1&wat_url=_tme-wrk_%2Ftme_new_render3d.png&wat_scale=100p&ci_sign=2d1584be91947c00a931f96d8b512f187d375fb2" width="140" alt="MCP6002 DIP-8"><br>$0.41 each<br>[TME product page](https://www.tme.com/us/en-us/details/mcp6002-i_p/tht-operational-amplifiers/microchip-technology/) · [Datasheet](https://ww1.microchip.com/downloads/aemDocuments/documents/MSLD/ProductDocuments/DataSheets/MCP6001-1R-1U-2-4-1-MHz-Low-Power-Op-Amp-DS20001733L.pdf) | • Runs from 1.8 to 6 V, so it can sit on the Nano's 3.3 V rail and its output can never exceed what the ADC allows<br>• Rail-to-rail input and output<br>• 1 MHz bandwidth, plenty for a force signal<br>• Dual package gives a spare amplifier for a second filter stage<br>• Very low supply current<br>• Very inexpensive | • Only 1 MHz, not suited to later high-speed ideas<br>• Low output drive, fine for an ADC but not for loads<br>• Limited to 6 V maximum supply |
| **Option 2. TI LM358P dual op-amp, DIP-8**<br><img src="https://ce8dc832c.cloudimg.io/v7/_cdn_/E9/D2/B0/00/0/732574_1.jpg?width=640&height=480&wat=1&wat_url=_tme-wrk_%2Ftme_new_render3d.png&wat_scale=100p&ci_sign=2d1584be91947c00a931f96d8b512f187d375fb2" width="140" alt="LM358P DIP-8 package"><br>$0.28 each<br>[TME product page](https://www.tme.com/us/en-us/details/lm358p/tht-operational-amplifiers/texas-instruments/) · [Datasheet](https://www.ti.com/lit/ds/symlink/lm358.pdf) | • Lowest price and widely available<br>• Input range includes ground, convenient for single-supply sensing<br>• 3 to 32 V supply range is flexible | • Output cannot swing within about 1.5 V of its positive supply, so on 3.3 V it only reaches about 1.8 V, which wastes ADC range<br>• On the 5 V rail the output can reach about 3.5 V, above the Nano's 3.3 V limit, so it would need a clamp<br>• Only 0.7 MHz bandwidth for the legacy part<br>• Higher quiescent current than the MCP6002 |
| **Option 3. Microchip MCP6022-I/P dual op-amp, DIP-8**<br><img src="https://ce8dc832c.cloudimg.io/v7/_cdn_/E9/D2/B0/00/0/732574_1.jpg?width=640&height=480&wat=1&wat_url=_tme-wrk_%2Ftme_new_render3d.png&wat_scale=100p&ci_sign=2d1584be91947c00a931f96d8b512f187d375fb2" width="140" alt="MCP6022 DIP-8 package"><br>$1.70 each<br>[TME product page](https://www.tme.com/us/en-us/details/mcp6022-i_p/tht-operational-amplifiers/microchip-technology/) · [Datasheet (MCP6021/2/3/4 family)](https://ww1.microchip.com/downloads/aemDocuments/documents/MSLD/ProductDocuments/DataSheets/MCP6021-Data-Sheet-DS20001685.pdf) | • 2.5 to 5.5 V supply, rail-to-rail, works on 3.3 V<br>• 10 MHz bandwidth leaves room for faster filtering<br>• Lower noise for small signals | • About four times the price of the MCP6002<br>• Draws more supply current<br>• Bandwidth is more than a force signal needs |

**Choice: Option 1, MCP6002-I/P [Pressure sensor subsystem]**

**Rationale:** The key constraint is that the Nano's ADC only accepts up to 3.3 V. The MCP6002 runs directly from the 3.3 V rail with rail-to-rail output, so its output physically cannot exceed the ADC range, and no clamp is needed. The LM358P cannot give a useful swing at 3.3 V, and on 5 V it could overdrive the input. The MCP6022's extra bandwidth does not help a slowly varying force signal. The spare amplifier in the package can be used as a comparator to produce a digital contact flag, which covers the "analog / digital" pressure read in the software flow, or as a second filter stage if the FSR signal is noisy.

---

### 2.5 [Power subsystem] 5 V linear voltage regulator

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. ST L7805CV 5 V regulator, TO-220**<br><img src="https://www.jameco.com/Jameco/Products/ProdImag/51262.jpg" width="140" alt="L7805CV"><br>$0.65 each<br>[Jameco product page](https://www.jameco.com/z/L7805CV-STMicroelectronics-Voltage-Regulator-5-Volt-1-5-Amp-TO-220_51262.html) · [ST datasheet](https://www.st.com/resource/en/datasheet/l78.pdf) | • Up to 1.5 A with internal current limiting and thermal shutdown<br>• Specified for 7 to 18 V input, which brackets the 9 V adapter<br>• Output capacitor only needed for transient response, so it is forgiving<br>• Very inexpensive and easy to source<br>• TO-220 takes a heatsink if needed | • Dropout of about 2 V at 1 A, so it needs more than 7 V in<br>• Quiescent current of about 8 mA<br>• Dissipates (9 V − 5 V) × load, which is about 0.9 W at my worst case<br>• Thermal resistance of about 50 °C/W without a heatsink |
| **Option 2. TI LM2940CT-5.0 low-dropout 5 V regulator, TO-220**<br><img src="https://mm.digikey.com/Volume0/opasdata/d220001/medias/images/3642/296%7ET03B%7ENDE%7E3.JPG" width="140" alt="LM2940CT-5.0"><br>$1.94 each<br>[Digi-Key product page](https://www.digikey.com/en/products/detail/texas-instruments/LM2940CT-5-0-NOPB/32575) · [TI datasheet](https://www.ti.com/lit/ds/symlink/lm2940.pdf) | • Dropout of about 1 V at 1 A, so it still regulates if the supply sags<br>• 1 A output, up to 26 V input<br>• Same TO-220 footprint as Option 1 | • Higher quiescent current (about 15 mA)<br>• More than twice the price of the L7805CV<br>• Low-dropout designs are more sensitive to output capacitor choice, so the datasheet capacitor requirement must be followed<br>• Dropout advantage is unused with a regulated 9 V supply |

**Choice: Option 1, L7805CV [Power subsystem]**

**Rationale:** The input is a regulated 9 V adapter, so there is 4 V of headroom over the L7805's 2 V dropout and low dropout buys nothing. My worst-case 5 V load with margin is about 0.22 A, giving about 0.87 W dissipated and a junction rise of about 43 °C at 50 °C/W, which is acceptable without a heatsink (see the [Power Budget](../05-Power-Budget/Power-Budget.md)). It is the least expensive option, forgiving on capacitors, and has built-in current limiting and thermal shutdown.

---

### 2.6 [Power subsystem] External 9 V power source

USB power packs are not allowed for this project, so all options are wall adapters.

| Solution | Pros | Cons |
|---|---|---|
| **Option 1. Adafruit 9 V 1 A regulated adapter, 5.5 × 2.1 mm, center-positive**<br><img src="https://www.adafruit.com/images/480x360/63-07.jpg" width="140" alt="Adafruit 9V 1A"><br>$8.95 each<br>[Adafruit product page](https://www.adafruit.com/product/63) | • Regulated output<br>• Barrel size and positive tip are stated on the page<br>• UL listed<br>• Lowest price | • 1 A is not enough: my worst-case need with margin is about 1.02 A, so remaining current would be negative<br>• No headroom for added sensors later |
| **Option 2. Pololu 9 V 1 A regulated adapter, 5.5 × 2.1 mm, center-positive**<br><img src="https://a.pololu-files.com/picture/0J7885.1200x627.jpg?8f2dfc183198b0004956b0dbd969b6a8" width="140" alt="Pololu 9V 1A"><br>$11.11 each<br>[Pololu product page](https://www.pololu.com/product/1470) | • Regulated, polarity stated, 1.5 m cord<br>• Light, small body<br>• Reliable supplier | • Same 1 A limit fails the budget<br>• Most expensive of the three |
| **Option 3. SparkFun 9 V 2 A regulated adapter, 5.5 × 2.1 mm**<br><img src="https://www.sparkfun.com/media/catalog/product/cache/6b78ac9ed927a3c2db42a3c84dab4ce5/3/1/31173-Power-Supply-Feature_2.jpg" width="140" alt="SparkFun 9V 2A"><br>$10.95 each<br>[SparkFun product page](https://www.sparkfun.com/wall-adapter-power-supply-9v-2a-barrel-jack.html) · [Datasheet](https://cdn.sparkfun.com/assets/9/4/e/b/9/9V__2A__US__Datasheet.pdf) | • 2 A leaves about 980 mA of margin after my 25% safety factor<br>• Regulated output<br>• Room for stall-current surges and later changes | • Plug polarity is not stated on the product page, so it must be checked before connecting<br>• Larger body than the 1 A adapters |

**Choice: Option 3, SparkFun 9 V 2 A adapter [Power subsystem]**

**Rationale:** The budget (see [Power Budget](../05-Power-Budget/Power-Budget.md)) needs about 1.02 A from the supply including the 25% margin, which fails both 1 A adapters and passes the 2 A unit with about 0.98 A to spare. Before using it, I will check the polarity in its datasheet and with a multimeter, since the barrel jack and reverse-polarity protection assume center-positive.

---

## 3. Checklist

- [x] At least 3 candidates per peripheral (regulator has 2, as the assignment allows)
- [x] Pictures and links present (prices as of 2026-10-19)
- [x] Multiple pros and cons per option
- [x] Rationale for each selection
- [x] Pin table included (draft, to be matched to the team block diagram)
- [ ] Power budget: see [Power Budget](../05-Power-Budget/Power-Budget.md)
