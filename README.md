# dESPcent

**dESPcent** is an experimental port of the original **Descent** engine to the **ESP32-S3**.

The goal is simple: find out just how much mid-1990s PC game an ESP32-S3 can actually run when we're executing the game natively rather than emulating the PC underneath it.

This project is currently targeting an **Elecrow CrowPanel 7" ESP32-S3 HMI** with an 800×480 display, 8 MB PSRAM, and 4 MB flash.

## Goals

- Run the original Descent engine natively on ESP32-S3
- Preserve the original fixed-point software-rendering architecture where practical
- Avoid unnecessary desktop compatibility layers and dependencies
- Use PSRAM carefully rather than assuming desktop-scale memory
- Support SD-card game data
- Support gamepad controls
- Provide sound and music
- Render internally at a resolution appropriate for the ESP32-S3
- Scale the resulting framebuffer efficiently to the 800×480 display
- Remain recognizable as the original Descent engine rather than becoming a ground-up reimplementation

## Display Strategy

The 800×480 panel is the **output resolution**, not necessarily the rendering resolution.

The original Descent renderer was designed for considerably more constrained hardware than a modern PC, making its software renderer and fixed-point mathematics particularly interesting on a microcontroller.

Initial targets will likely include resolutions such as:

- 320×200
- 320×240
- 384×240

A 384×240 framebuffer can be scaled 2× to **768×480**, leaving only narrow side borders on the 800×480 display while requiring the engine to render less than one quarter of the panel's native pixel count.

## Target Hardware

Initial development target:

**Elecrow CrowPanel 7" ESP32 Display**

- ESP32-S3-WROOM-1-N4R8
- Dual-core Xtensa LX7
- 4 MB Flash
- 8 MB PSRAM
- 800×480 TFT LCD
- Capacitive touchscreen
- GPIO, UART and I²C expansion
- Battery support

Other ESP32-S3 hardware may be supported later where practical.

## Development Philosophy

Accuracy and functionality come first.

The initial objective is not to rewrite or heavily simplify Descent until it fits. Instead, we'll establish a working baseline, identify the actual CPU, memory, rendering, and I/O bottlenecks, and optimize based on measurements.

If something is slow, we want to know **why** it's slow.

If something has to be changed for the ESP32-S3, that change should have a measurable reason for existing.

## Current Status

**Very early development.**

Right now, dESPcent is an experiment:

> Can an ESP32-S3 run a useful native port of Descent?

We don't know yet.

That's the fun part.

## Game Data

dESPcent does **not** intend to distribute the original Descent game assets.

Users will need to provide compatible game data from their own copy of Descent.

## Source and Licensing

Descent's original source code was publicly released by Parallax Software, and subsequent projects such as D1X and DXX-Rebirth have continued development of the engine.

dESPcent will retain applicable copyright notices and licensing requirements from the source code on which it is based.

The exact source baseline and resulting licensing documentation will be established as the port is brought up.

**Descent** is a trademark of its respective owners.

dESPcent is an independent open-source project and is not affiliated with or endorsed by the owners of the Descent intellectual property.

## Why?

Because an ESP32-S3 running Descent would be ridiculous.

And because we want to know if it can.
