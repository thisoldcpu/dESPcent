# dESPcent

**dESPcent** is an experimental native port of the original **Descent** engine to the **ESP32-S3**.

The goal is simple: find out how much mid-1990s PC game an ESP32-S3 can run when it executes the engine directly, without emulating the PC underneath it.

The current target is an **Elecrow CrowPanel 7-inch HMI, V3.0**, with an **ESP32-S3-WROOM-1-N4R8**, an **800×480 RGB display**, **8 MB PSRAM**, and **4 MB flash**.

## Current Status

**Gameplay is working on real ESP32-S3 hardware, demos run to completion, and a player can play a full run now.**

*Last updated: September 26, 2026. Experimental development build.*

**Main menu, item by item:**

- [x] New Game
- [ ] Load Game...
- [ ] Multiplayer...
- [x] Options...
- [ ] Change Pilots...
- [x] View Demo...
- [x] High Scores
- [x] Credits
- [x] Quit
  - [ ] Load Level...
  - [ ] Play Song

dESPcent now runs the original engine through startup, menus, mission briefings, and into working gameplay. Demo playback also runs to completion. Current work is focused on two things: tracking down the project's last standing heap-corruption bug (internally tracked as "Issue 3"), and systematically checking the remaining screens and adapting their display updates and wait loops to the ESP32 graphics backend and FreeRTOS scheduler.

The current hardware build has demonstrated:

- ESP-IDF v6.0.1 startup at 240 MHz with 8 MB PSRAM and 4 MB flash.
- FAT SD-card mounting and game-data discovery.
- 800×480 RGB panel initialization.
- GT911 capacitive touch input.
- BLE connection to an Xbox Wireless Controller.
- Animated hardware/data POST with SD verification, battery state, and touch-to-continue.
- Readable `DESCENT.HOG` and `DESCENT.PIG` data and recognition of the known early registered D1 PIG layout.
- Native entry into the original Descent `INFERNO` startup path.
- The registered v1.5 engine banner.
- Original palette, font, bitmap, sound, polygon-model, 3D, and texture-cache initialization.
- Interplay, Parallax Software, and Descent title screens rendered directly on the ESP32-S3 panel.
- Original menu logic running with controller navigation.
- New Game flow progressing through mission and difficulty selection.
- Mission briefing backgrounds, text, timing, and page transitions.
- Controller input translated into the legacy Descent menu/title/briefing input paths.
- Transition from menus and briefings into the actual game loop.
- In-game cockpit/HUD and level rendering during working gameplay.
- Active player and robot physics execution.
- Demo playback running to completion.

### Current focus: chasing the last heap-corruption bug ("Issue 3")

The project's long-standing intermittent PSRAM heap-corruption crash, tracked internally as "Issue 3," saw its first real progress today. A byte-level guard system was added around every canvas allocated by `gr_create_canvas()` — the 320×200 offscreen render target, the HUD gauges, the weapon reticle, and others — on top of the existing guard around the main indexed framebuffer. Combined with two other fixes made the same day (a double-free in `IFF.cpp`'s bitmap loader that had been silently masked for a long time by an unsafe `free()` macro in this port's own `LIB/MEM.H`, and the removal of that macro itself), both View Demo and a full New Game session ran for hundreds of frames with no crash — a first for this project's debugging history. Both runs were stopped manually, not crashed.

The new canvas guards also caught something during those same runs: a single stray byte written exactly one byte before the start of the offscreen render target's pixel data, at the same offset every time, though at different frame counts and player positions across runs. That's the signature of a deterministic off-by-one — most likely an unclamped coordinate at the edge of a scanline or a blit run — rather than random corruption, and in both runs it didn't happen to land anywhere load-bearing. It's the first byte-level lead this investigation has had. Tracing the exact write site is next; leading suspects are the 3D scanline rasterizer and the cockpit's `gr_ibitblt()` run-list blitter, which has no bounds checking by design.

Separately, a menu-side scheduler bug was found and fixed: reaching "View High Scores" from the main menu without a new high score to show had no yield point at all in its input-polling loop, which pegged CPU0 and tripped the watchdog every five seconds. That fix is committed but not yet confirmed on hardware.

Some original DOS screen loops still need explicit display updates and scheduler yields of their own. On this port, drawing into the indexed screen buffer must be followed by `gr_present()` to update the panel, and polling/timing loops need `vTaskDelay(...)` so other FreeRTOS tasks can run. The credits screen was fixed this way earlier and still needs verification on hardware; remaining screens are being checked systematically for the same omission.

Working gameplay, completed demos, and now hundreds-of-frames-stable runs are milestones, not a claim that every screen, mission, or feature has been validated.

## Startup and Rendering Milestones

<p align="center">
  <img width="900" alt="dESPcent startup POST verifying Descent data on the CrowPanel" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_post_verifying.jpg?raw=true" />
</p>

<p align="center">
  <em>The animated POST verifies SD data while hardware status remains visible, including live battery state.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent startup POST complete and ready to launch" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_post_verifed.jpg?raw=true" />
</p>

<p align="center">
  <em>Verification complete and ready to launch. Touch input hands control off to the original Descent startup sequence.</em>
</p>

<p align="center">
  <img width="900" alt="Interplay logo rendered by dESPcent on the CrowPanel" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_interplay_logo.jpg?raw=true" />
</p>

<p align="center">
  <em>The software renderer outputs the Interplay logo directly to the ESP32-S3 RGB panel.</em>
</p>

<p align="center">
  <img width="900" alt="Parallax Software logo rendered by dESPcent on the CrowPanel" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_parallax_logo.jpg?raw=true" />
</p>

<p align="center">
  <em>The Parallax Software logo sequence rendered natively on the CrowPanel.</em>
</p>

<p align="center">
  <img width="900" alt="Descent title screen rendered by dESPcent on the CrowPanel" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_descent_logo.jpg?raw=true" />
</p>

<p align="center">
  <em>The original Descent title screen as the native startup sequence continues on the ESP32-S3.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent DOS-style hardware setup screen" src="https://github.com/user-attachments/assets/425561aa-bea6-4958-85b6-7b98dcaeb72d?raw=true" />
</p>

<p align="center">
  <em>A DOS-style hardware setup screen inspired by the original Descent ASCII setup program.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent pilot-name screen on ESP32-S3 hardware" src="https://github.com/user-attachments/assets/e19f119b-408c-474e-84ac-e4683a3767ea" />
</p>

<p align="center">
  <em>The original pilot-name flow running natively on the ESP32-S3. Existing configuration data is already being read by the engine.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent joystick calibration screen on ESP32-S3 hardware" src="https://github.com/user-attachments/assets/4404d314-385b-4ad4-b716-1e80a82e5c5e" />
</p>

<p align="center">
  <em>Descent's original joystick-calibration path, reached through the ported controller/input layer.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent main menu on ESP32-S3 hardware" src="https://github.com/user-attachments/assets/a7188166-6f86-40c6-bd52-1e05daa156ea" />
</p>

<p align="center">
  <em>The original Descent main menu, fully visible and controller-navigable on the CrowPanel.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent mission briefing on ESP32-S3 hardware" src="https://github.com/user-attachments/assets/e9dd06a0-48b5-4424-abcb-f401a3160feb" />
</p>

<p align="center">
  <em>The mission briefing system running on hardware, including original backgrounds, palette effects, timed text, and page flow.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent character briefing screen on ESP32-S3 hardware" src="https://github.com/user-attachments/assets/02d75b06-1713-40d5-bdb0-3d621c8f3e4f" />
</p>

<p align="center">
  <em>A later briefing page with character artwork and scrolling mission text. The original 320×200 presentation is scaled 2× to 640×400 and centered on the 800×480 panel.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent first in-game cockpit frame on ESP32-S3 hardware" src="https://github.com/user-attachments/assets/942f31d8-57b0-4d3b-bdc2-a310245a6641" />
</p>

<p align="center">
  <em>First in-game cockpit frame on the ESP32-S3. The game loop is active, player and robot physics are running, and the original software renderer is producing the scene natively.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent first in-game cockpit frame on ESP32-S3 hardware" src="https://github.com/user-attachments/assets/021ca1c8-2569-4225-b3e5-d951bc0e18df" />
</p>

<p align="center">
  <em>First demo playback cockpit frame on the ESP32-S3. The game loop is active and playback is running.</em>
</p>

## Goals

<p align="center">
  <strong>Original engine. Native ESP32-S3. No PC underneath it.</strong><br>
  <sub>Preserve Descent's behavior and architecture, replace only the hardware and operating-system contracts that no longer exist.</sub>
</p>

<br>

<table>
<tr>
<td width="33%" valign="top">

### Preserve

Keep the original fixed-point mathematics, software renderer, game logic, data formats, and engine contracts wherever practical.

This is a **port of Descent**, not a ground-up recreation.

</td>
<td width="33%" valign="top">

### Replace

Translate the machine underneath it:

- DOS and BIOS services → ESP-IDF
- VGA framebuffer/DAC → indexed framebuffer + RGB panel
- DOS input → BLE HID / touch
- DOS audio hardware → I²S
- local filesystem → SD/FAT

</td>
<td width="33%" valign="top">

### Measure

Let the ESP32-S3 tell us where the limits are.

Use internal SRAM and PSRAM deliberately, preserve correctness first, and optimize rendering, memory, audio, and I/O from actual measurements rather than assumptions.

</td>
</tr>
</table>

> **Porting rule:** if the original engine has a contract, reproduce that contract before changing the engine around it.

The practical target is the original **320×200 indexed renderer**, presented at **2× nearest-neighbor scale as 640×400** and centered on the CrowPanel's 800×480 RGB display.

Controller input, digital sound, SD-based game data, and the original menu/briefing/gameplay flow are all intended to remain recognizably Descent.

---

## Target Hardware

<p align="center">
  <strong>Current reference platform</strong><br>
  <sub>Elecrow CrowPanel 7-inch HMI V3.0 · ESP32-S3 · 8 MB PSRAM · 800×480 RGB</sub>
</p>

### Core Platform

| Component | Specification |
| --- | --- |
| **Board** | Elecrow CrowPanel 7-inch HMI, V3.0 |
| **Processor** | Dual-core ESP32-S3, configured at 240 MHz |
| **Memory** | Internal SRAM + 8 MB external octal PSRAM at 80 MHz |
| **Flash** | 4 MB QIO at 80 MHz |
| **Storage** | FAT-formatted SD card over SPI |

### Display & Input

| Component | Specification |
| --- | --- |
| **Display** | 800×480 RGB panel, RGB565 scanout |
| **Engine framebuffer** | Original 320×200 indexed-color surface |
| **Presentation** | 2× nearest-neighbor → 640×400, centered at 80×40 |
| **Touch** | GT911 capacitive controller via PCA9557 on V3.0 |
| **Controller** | BLE Xbox Wireless Controller; decoder targets model 1914 |

### Audio & Prototype Power

| Component | Specification |
| --- | --- |
| **Audio** | I²S digital output through NS4168 amplifier |
| **Prototype battery** | MakerFocus 3.7 V, 3,700 mAh |
| **Battery gauge** | Adafruit LC709203F over I²C |

<p align="center">
  <img width="900" alt="Rear of the dESPcent prototype showing battery and fuel gauge hardware" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_proto_0.1_rear.jpg?raw=true" />
</p>

<p align="center">
  <em>Rear of the current prototype, showing the MakerFocus 3,700 mAh battery and Adafruit LC709203F fuel gauge.</em>
</p>

The battery system is part of the current prototype rather than a requirement of dESPcent. Runtime on battery has not yet been characterized.

### Peripheral Connections

| Interface | GPIO |
| --- | ---: |
| **SD** — MOSI / MISO / SCLK / CS | 11 / 13 / 12 / 10 |
| **I²C** — SDA / SCL | 19 / 20 |
| **I²S** — DOUT / LRCLK / BCLK | 17 / 18 / 42 |
| **LCD backlight** | 2 |

<p align="center">
  <sub>The port is currently board-specific. Additional ESP32-S3 targets can come later once the reference platform is stable.</sub>
</p>

## Memory and Display Strategy

Having 8 MB of PSRAM does not mean that 8 MB is available to the game. Large engine arrays, firmware instructions and read-only data, display buffers, Bluetooth allocations, graphics state, sound data, and loaded polygon models all compete for that memory. Internal RAM is still required for task stacks, DMA-capable allocations, and other ESP-IDF resources.

Current major graphics allocations include:

| Buffer | Size |
| --- | ---: |
| One 800×480 RGB565 panel framebuffer | 768,000 bytes |
| 320×200 indexed Descent framebuffer | 64,000 bytes |
| Two internal RGB DMA bounce buffers, 10 lines each | 32,000 bytes total |

The original Descent renderer remains palette-indexed at its native **320×200** resolution. On DOS/VGA hardware, the framebuffer contained 8-bit palette indices and the VGA DAC performed color lookup during scanout.

The ESP32 has no VGA DAC, so dESPcent preserves the original indexed framebuffer and maintains a software equivalent of the DAC palette. At presentation time, the 320×200 image is converted to RGB565, scaled **2× nearest-neighbor to 640×400**, and centered at **80×40** inside the panel's 800×480 framebuffer.

This keeps the original renderer, UI layout, fonts, cockpit, menus, and briefing screens operating in their native coordinate system while moving panel-specific scaling entirely into the ESP32 graphics backend.

The palette state also remains faithful to the original architecture: the engine's retained palette is kept separate from the currently displayed software-DAC palette.

The current asset load reaches approximately:

- **1.29 MB** reserved for `SoundBits`.
- **2.00 MB** reserved for the bitmap cache.
- **78 of 85** polygon-model slots populated during the current registered-data startup path.

Those numbers are diagnostic snapshots, not fixed requirements.

At the configured 15 MHz pixel clock and 928×525 total timing, the calculated panel refresh rate is approximately **30.79 Hz**. Active RGB565 scanout payload is approximately **23.65 MB/s**, before rendering writes and other memory traffic. This is a display timing calculation, not a measured game frame rate.

## Controls

BLE controller support has moved beyond connection testing.

The current Xbox controller path is integrated into Descent's joystick/menu contracts and has been demonstrated navigating menus and advancing title/briefing screens. Current logical mappings include:

- **A / Start** → Enter/confirm in menu-style interfaces.
- **B** → Escape/back.
- **D-pad** → menu navigation.

The engine's joystick configuration and calibration paths are active, and gameplay is now working. Full validation of 6DOF control, axis behavior, deadzones, and final bindings remains part of ongoing hardware testing.

The GT911 also has a touch-backed mouse implementation. Touch currently participates in startup handoff; broader menu/game use is still secondary to the controller path.

## Audio

Digital sound output is working on hardware.

The original sound tables and sample data load successfully. Recent startup logs report 108 sound slots in use and roughly 404 KB of actual sample payload selected from the larger sound reservation.

The ESP32 I²S backend is active through the CrowPanel's NS4168 amplifier and external speakers. The Descent Sound FX volume slider has been verified to play its test sound at the selected volume levels, confirming end-to-end sample playback and volume scaling through the ESP32 audio path.

Broader in-game sound coverage still needs validation as gameplay progresses, but the digital sound backend itself is no longer merely a placeholder or untested port.

## Game Data

Provide compatible `DESCENT.HOG` and `DESCENT.PIG` files from your own copy of Descent. Original game assets are not part of the intended project distribution.

Place both files together in the SD card root or in a directory named `DESCENT`:

```text
SD card/
└── DESCENT/
    ├── DESCENT.HOG
    └── DESCENT.PIG
```

The firmware mounts the card at `/sdcard`. A complete readable pair in the root takes precedence over `/DESCENT`; `/DESCENT_AUTO` is the fallback for automatically installed data.

POST reports file readability, known data fingerprints, and archive-layout information. A `PASS` for a readable, nonempty file is not proof that every asset is valid or that an entire release is playable. The engine's v1.5 banner and the detected data release are separate identifiers.

The currently exercised data path uses an early registered D1 PIG layout. Support for that layout required porting the original bitmap/game-definition reader rather than substituting a newer asset format.

### Optional ISO installation

Instead of copying the two archives yourself, place one supported ISO image in the SD root or `/DESCENT`. If a readable loose pair is already available, it is used without extraction. Otherwise, the installer looks for both archives together inside the image, stages and verifies them, and promotes the result to `/DESCENT_AUTO`.

Existing game-data directories are not overwritten. The ISO can be removed after successful installation; runtime uses the extracted archives. Compressed installers and raw BIN/CUE images are not supported by this reader.

## Known Incomplete Areas

The project has crossed enough startup milestones that the remaining work is increasingly normal game-port work rather than bootstrapping.

Current known incomplete areas include:

- Root-causing the remaining "Issue 3" off-by-one write into the offscreen canvas. The corruption is now localized to a specific 1-byte, per-frame signature; the exact write site is not yet traced.
- Hardware confirmation of the `scores_view()` scheduler-yield fix (the "View High Scores" watchdog timeout).
- A tree-wide sweep for other instances of the double-free pattern found in `IFF.cpp` — an explicit `free()` immediately followed by a cleanup helper that also frees the same buffer.
- Remaining screen loops that need explicit `gr_present()` calls or FreeRTOS scheduler yields.
- Hardware verification of the credits presentation fix and other screen-specific changes.
- Final in-game controller axis behavior and bindings.
- Full audio playback validation.
- Player/config persistence cleanup. A present `.PLR` file does not yet always produce the expected Select Pilot flow, and controller-choice persistence is still being restored.
- Display tearing/flicker associated with the current single-buffer memory strategy.
- Performance characterization under real gameplay load: multiple robots, projectiles, collision checks, AI, sound mixing, and sustained rendering.
- Network support.
- Further DOS shim cleanup and consolidation.

## Development Philosophy

Accuracy and functionality come first.

The port is intentionally source-faithful: preserve the original engine contracts where practical, replace only the hardware/OS services that do not exist on ESP32, establish a correct baseline, and optimize from measurements rather than guesses.

That distinction has become increasingly important as more of the original engine comes alive. VGA-era code often relied on hardware behavior that was implicit on DOS machines: indexed framebuffer scanout, DAC palette state, timer behavior, and direct-input assumptions. The ESP32 replacements need to reproduce those contracts without silently changing what the engine thinks those systems mean.

If something is slow, we want to know **why**. If something changes for the ESP32-S3, there should be a concrete reason for the change.

This is an ESP-IDF CMake project. Current hardware runs use **ESP-IDF v6.0.1** with the `esp32s3` target. The root `sdkconfig`, `sdkconfig.defaults`, and `partitions.csv` describe the active configuration. The custom partition table provides a single factory application partition within 4 MB flash; it does not provide OTA slots.

The files under [docs](docs) record individual porting steps and checks. Some describe earlier snapshots and should not be read as current feature status.

## Source and Licensing

The source baseline is the original **Descent 1 v1.5** release, rather than D1X or DXX-Rebirth. Platform-specific code is being adapted or replaced for ESP32-S3 while retaining the original engine structure and notices.

The ported Parallax source files carry a license notice restricting use to non-commercial, royalty- or revenue-free purposes. The project should not be described as having a blanket permissive open-source license. Independently authored dESPcent software is offered under MIT within the scope defined by the root [LICENSE](LICENSE). Parallax adaptations and third-party components retain their existing terms. See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) and [LICENSES](LICENSES) for the identified upstream notices and full license texts.

Game data is separate from the source release and is not made freely distributable by the availability of engine source.

**Descent** is a trademark of its respective owners. dESPcent is an independent project and is not affiliated with or endorsed by those owners.

## Why?

Because an ESP32-S3 running Descent would be ridiculous.

And now gameplay works, demos run to completion, and it's starting to hold up under sustained play.

Next: track down the last heap-corruption bug, finish the remaining screens, refine the controls, and measure how it holds up in the mine.
