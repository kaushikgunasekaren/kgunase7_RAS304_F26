---
title: Individual Block Diagram
tags:
- block diagram
- linear actuator
- accelerometer
---

> **Update (23 Nov 2026):** this page shows the design as submitted. In [BOM revision 2](../03-BOM/BOM.md#revision-2-changes-2026-11-23) the motor became a lab Pololu #3067 / DORHEA N20 6 V gearmotor (no lead screw), the slide pot became a 10k rotary pot (P-3), and the IMU became an off-board ADXL345 module on I2C (U5 not fitted). The circuit and PCB are otherwise unchanged.


## Overview

This page shows the electrical architecture of my subsystem for Team 102's Modular Spider Leg. In the team's hub-and-spoke layout, my board is the spoke on **hub connector 3**. It drives the leg's small **linear actuator** (the actuator) and measures leg motion with an **accelerometer** for jerk and impulse sensing (the sensor). Commands come from Preston's hub board over the ribbon cable.

The design follows the [RAS 304 project requirements](https://ras-embedded-systems.pages.dev/304/course-info/project-description/):

* PIC18F57Q43 Curiosity Nano as the microcontroller.
* A 9 V barrel jack feeding a 5 V **linear** regulator, with a fuse and a power-status LED. (My board adds a second linear regulator, 6 V, for the motor only.)
* A 2x4 male IDC header using the class ribbon pinout.
* No daughter boards: every peripheral, including the accelerometer, is placed directly on this PCB.

Parts were also chosen to stay within the team's $70&ndash;80 total budget.

The block diagram shows:

* **Power levels:** my own 9 V adapter &rarr; barrel jack &rarr; 1 A fuse &rarr; 1N5819 reverse-polarity diode &rarr; two linear regulators. The **LM317TG (set to 5.9 V)** feeds only the motor driver and the 6 V actuator motor. The **L7805CV (5 V)** feeds the Curiosity Nano and the power LED. The Curiosity Nano's on-board regulator supplies 3.3 V to the accelerometer, the position potentiometer and the current-sense op-amp.
* **Actuator:** a DFRobot FIT0577 N20 gear motor with an M3 lead screw, driven by a TI DRV8833 H-bridge. A 0.33 &Omega; current-sense resistor sets a 0.61 A current limit, and an MCP601 amplifier (gain 11) scales its voltage for the ADC so the PIC can detect a stall.
* **Position feedback:** a Bourns slide potentiometer whose slider moves with the actuator's lead-screw nut.
* **Sensor:** an Analog Devices ADXL343 3-axis accelerometer chip on SPI, with a data-ready interrupt.
* **Debug:** two debug LEDs and a debounced push button on spare GPIO.
* **Team connection:** an 8-pin ribbon cable to Preston's hub board. It carries position commands in and jerk/impulse data out over UART.

## Block Diagram

![Individual block diagram](individual-block-diagram.png)  
**Figure 1:** Individual block diagram for the linear actuator and accelerometer subsystem.

Source files: [draw.io source (ZIP)](individual-block-diagram-source.zip) · [draw.io source file](individual-block-diagram.drawio)

## How the Block Diagram Was Structured

The diagram is organized left to right by **supply domain** (9 V &rarr; 6 V &rarr; 5 V &rarr; 3.3 V), so a reader can follow power from the wall adapter to every load, and each dashed box states its voltage, regulator and maximum current. The microcontroller sits in the middle with each peripheral (PWM, ADC, SPI, UART, digital I/O) drawn as its own white block, so every signal line shows its type, pin count and the PIC pin it uses. Actuator parts are grouped on the left and sensing parts on the right, and the ribbon connector to Preston's hub is at the bottom right, matching the team block diagram.

Since the first version, two changes came from the power budget and feedback: the motor moved from the 5 V rail to its own **6 V regulator** (the 6 V motor was running under-voltage and the 5 V regulator was overheating), and an **op-amp** was added after the current-sense resistor because 0&ndash;0.2 V used only a small slice of the ADC range.

## How the Design Meets the Product Requirements

| Requirement (team) | How this board meets it |
|---|---|
| **2.2** Standard communication protocols | UART to the hub over the ribbon; SPI to the accelerometer |
| **2.3** Modular software: motion, sensing, communication separated | Separate peripherals for motion (PWM + position ADC), sensing (SPI + interrupt) and communication (UART) |
| **4.4 / 5.2** Off-the-shelf parts and connectors | Catalog parts only; motor and slide pot plug into headers so they can be replaced |
| **6.1** Current limiting or stall detection on actuators | DRV8833 0.61 A current limit, amplified current sense on RA0, and nFAULT on RB1 |
| **6.3** Over-voltage and over-current protection | 1 A fuse, reverse-polarity diode, regulators with internal current limit and thermal shutdown |
| **6.4** Fail-safe stop if communication is lost | nSLEEP (RD1) has a pull-down, so the motor driver stays off while the PIC is in reset; firmware can also pull nSLEEP low to stop the motor if hub commands stop arriving |

## Major Components and Cost

| Function | Manufacturer | Part Number | Supply | Approx. Cost |
|---|---|---|---|---|
| Microcontroller | Microchip | PIC18F57Q43 Curiosity Nano (DM164150) | 5 V (VBUS) | Course kit |
| Linear actuator | DFRobot | FIT0577 N20 gear motor, 6 V, 98 RPM, with M3 &times; 20 mm lead-screw shaft | 6 V | $9.00 |
| Motor driver (H-bridge) | Texas Instruments | DRV8833PWPR | 6 V (VM) | $3.02 |
| Position sensor | Bourns | PTA2043-2015CPB103 (10 k&Omega; slide pot, 20 mm travel) | 3.3 V | $1.78 |
| Accelerometer (bare chip) | Analog Devices | ADXL343BCCZ | 3.3 V | $4.02 |
| Current-sense amplifier | Microchip | MCP601-I/P | 3.3 V | $0.63 |
| Linear regulator (9 V &rarr; 5.9 V, motor) | onsemi | LM317TG (adjustable, set by 2 resistors) | 9 V in | $1.30 |
| Linear regulator (9 V &rarr; 5 V, logic) | STMicroelectronics | L7805CV | 9 V in | $1.14 |
| Reverse-polarity diode | onsemi | 1N5819RLG (6 on the board) | 9 V | $2.06 |
| Fuse (1 A, fast-acting, removable) | Littelfuse | 0217001.HXP + 2&times; Keystone 3517 clips | 9 V | TECH 162 |
| Barrel jack | CUI Devices | PJ-002A | 9 V | $0.66 |
| Power status LED (green) + 1 k&Omega; resistor | Kingbright | WP7113GD | 5 V | TECH 162 |
| 0.33 &Omega; 1 W sense resistor and 2.2 &micro;F capacitor | YAGEO / TDK | FMP100JR-52-0R33, FA16X7R1E225KNU06 | &mdash; | $0.88 |
| 2x4 IDC header and other capacitors and resistors | &mdash; | &mdash; | &mdash; | TECH 162 |
| 9 V wall adapter | &mdash; | &mdash; | &mdash; | Already owned |
| **Total for one board (before shipping)** | | | | **$24.49** |

The team budget is $70&ndash;80 for four boards, so an even share is $17.50&ndash;$20.00. At about $24.49 this board is above that range, mainly because of the motor and the protection parts, so the team total should be checked. The fuse, LED, header and common passives come from TECH 162, which does not count toward the budget. A plain 300 RPM N20 motor ($6.95) with a separate threaded rod and coupler would save about $2 but needs extra mechanical work. See the [Component Selection](../02-Component-Selection/Component-Selection.md) page.

## Power Domains

| Domain | Regulation | Max Current | Source | Loads |
|---|---|---|---|---|
| +9 V DC | Unregulated | 1.0 A | My 9 V wall adapter, through the barrel jack, a 1 A fuse and a 1N5819 diode | LM317TG and L7805CV inputs |
| +6 V DC (5.9 V) | Regulated (linear) | 1.5 A | onsemi LM317TG | DRV8833 (VM) and the N20 motor |
| +5 V DC | Regulated (linear) | 1.5 A | ST L7805CV | Curiosity Nano (VBUS, through JP1 and D3), power LED |
| +3.3 V DC | Regulated | 0.5 A | Curiosity Nano on-board regulator (VTG pin) | ADXL343BCCZ, slide potentiometer, MCP601 |

With the motor on its own regulator, the L7805CV carries only about 0.13 A and needs no heatsink. See the [Power Budget](../05-Power-Budget/Power-Budget.md) page for the current and heat calculations.

## Microcontroller Peripherals and Pins

| Peripheral | PIC18F57Q43 Pin(s) | Connected To | Signal (Type, Pins) |
|---|---|---|---|
| PWM (PWM1) | RC2, RD0 | DRV8833 AIN1, AIN2 | Analog (0&ndash;3.3 VDC PWM, 2 pins) |
| DO | RD1 | DRV8833 nSLEEP | Digital - Parallel (1 pin) |
| DI (interrupt-on-change) | RB1 | DRV8833 nFAULT | Digital - Parallel (1 pin) |
| ADC | RA0 (ANA0) | MCP601 output (amplified AISEN sense voltage) | Analog (0&ndash;2.2 VDC, 1 pin) |
| ADC | RA1 (ANA1) | Slide potentiometer wiper | Analog (0&ndash;3.3 VDC, 1 pin) |
| SPI (SPI1) | RC3 SCK, RC4 SDI, RC5 SDO, RD4 CS | ADXL343BCCZ | Digital - Serial (SPI, 4 pins) |
| DI (INT0) | RB0 | ADXL343BCCZ INT1 (data ready) | Digital - Parallel (1 pin) |
| UART (UART1) | RC7 RX, RC6 TX | Ribbon connector pins 1 and 2 | Digital - Serial (UART, 2 pins) |
| DO | RD2, RD3 | Debug LEDs | Digital - Parallel (2 pins) |
| DI | RD5 | Debug push button (debounced) | Digital - Parallel (1 pin) |
| USB | On-board debugger | PC (programming / debugging) | USB |

The DRV8833 drives the motor through AOUT1 and AOUT2: Analog (0&ndash;6 VDC PWM, 2 pins).

## Team Connection

My board connects to Preston's hub board (Leg Kinematics) through **hub connector 3**, using the class-standard 8-pin ribbon cable and a 2x4 male IDC header. **Each teammate's board is powered individually; no power is shared between boards.** My board runs from its own 9 V adapter, and the only connection to the hub is communication: UART TX and RX on pins 1 and 2. Pin 8 is a common ground, which carries no supply current but is needed so both boards agree on what a UART logic high and low are. No power pins are used on the ribbon.

| Pin | Type | Signal | Direction | MCU Pin |
|:---:|---|---|---|---|
| 1 | Digital | UART RX: position commands from the hub | Hub &rarr; my board | RC7 (U1RX) |
| 2 | Digital | UART TX: jerk / impulse data to the hub | My board &rarr; hub | RC6 (U1TX) |
| 3&ndash;5 | Digital | Not used | &mdash; | &mdash; |
| 6&ndash;7 | Analog | Not used | &mdash; | &mdash; |
| 8 | GND | Common ground | &mdash; | GND |

## Design Notes

* **Closed-loop position control:** the hub sends a target position over UART. The PIC compares it with the slide-pot reading on RA1, then drives the two PWM outputs (RC2, RD0) to move the lead screw forward or back.
* **Stall and fault protection (Req. 6.1):** current through the DRV8833 flows through a 0.33 &Omega; sense resistor on AISEN. An MCP601 amplifies that voltage by 11, and the PIC reads it on RA0 and stops the motor if the current stays high, which indicates a stall. The same resistor also sets the driver's built-in current limit at about 0.61 A (0.2 V &divide; 0.33 &Omega;), which keeps a motor fault within what my 1 A adapter can supply. The nFAULT pin (RB1) reports over-current or over-temperature, and the PIC then disables the driver through nSLEEP (RD1).
* **Jerk and impulse sensing:** the ADXL343 data-ready interrupt (RB0) triggers an SPI read of acceleration at up to 3.2 kHz. Jerk is found by differentiating acceleration, and impulse by integrating force (mass &times; acceleration) over the foot-contact time. The results are sent to the hub over UART.
* **Accelerometer soldering:** the ADXL343BCCZ is a 3 &times; 5 mm LGA-14 package. Because daughter boards aren't allowed, it is soldered directly to the PCB with a stencil and hot air or reflow.
* **Speed vs. cost:** a standard M3 lead screw moves 0.5 mm per turn, so the 98 RPM (60 RPM under load) FIT0577 gives about 0.5&ndash;0.8 mm/s. That is slow, but adequate for a small demonstration leg, and much cheaper than a commercial linear actuator with built-in feedback (about $80 or more).
