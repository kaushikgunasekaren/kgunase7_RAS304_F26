---
title: Welcome
tags:
- datasheet
- linear actuator
- accelerometer
---
<center>
<font size= "6">Kaushik Gunasekaren Datasheet</font><br>
as part of<br>
<font size= "8"> Modular Spider Leg</font><br>
for<br>
<font size= "5"> Team 102 </font><br>

**Submission: October 29, 2026**
</center>

## Introduction

This datasheet documents my individual subsystem board for RAS 304 (Embedded Systems). It walks through how the board was designed: its block diagram, the components chosen and why, the power budget, the schematic and the bill of materials.

### Project Summary

Team 102 is building a **modular robotic spider leg** for robotics hobbyists and students: a leg made from 3D-printed parts and off-the-shelf electronics that users can buy assembled, repair themselves, and wire into their own walking robots. The leg uses a hub-and-spoke design. Preston's hub board runs the leg kinematics and sends commands over ribbon cables to each teammate's board, and each board drives one part of the leg and reports back. Each board has its own power supply; the only connection between teammates' boards is the UART communication link.

My board, **Driver Nano 3**, is the spoke on hub connector 3. It drives **Linear Actuator B** (an N20 gearmotor turning an M3 lead screw) to the position the hub asks for, using a slide potentiometer for position feedback and motor-current sensing for stall protection. It also reads an **LIS3DH accelerometer** to measure the leg's jerk and impulse, which it sends back to the hub. More detail on the whole project is in the [Team 102 report](https://ras304-fall26-102.github.io/).

### My Contribution

* Linear actuator control: DRV8833 H-bridge with current limit, MCP601 current-sense amplifier, and closed-loop position from a slide potentiometer.
* Motion sensing: LIS3DH accelerometer on SPI for jerk and impulse data.
* Board power: 9 V input with fuse and reverse-polarity protection, a 6 V motor rail and a 5 V logic rail.
* UART link to the hub over the class-standard 8-pin ribbon cable.

### Navigating this datasheet

| Section | What you'll find |
|---|---|
| [Block Diagram](01-Block-Diagram/Block-Diagram.md) | Supply domains, every major part and the microcontroller pin each signal uses, and how the design meets the team requirements |
| [Component Selection](02-Component-Selection/Component-Selection.md) | Options compared for each major part, the choice and why, the MCC configuration, and cost |
| [BOM](03-BOM/BOM.md) | Every part on the board with part numbers and reference designators |
| [Schematics](04-Schematic/schematic.md) | The full KiCad schematic (image, PDF and project ZIP) and how it meets user needs |
| [Power Budget](05-Power-Budget/Power-Budget.md) | Current on each rail, regulator heat, and what the budget changed in the design |

Team report: [https://ras304-fall26-102.github.io/](https://ras304-fall26-102.github.io/)
