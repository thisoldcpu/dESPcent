# dESPcent

**dESPcent** is an experimental native port of the original **Descent** engine to the **ESP32-S3**.

The goal is simple: find out how much mid-1990s PC game an ESP32-S3 can run when it executes the engine directly, without emulating the PC underneath it.

The current target is an **Elecrow CrowPanel 7-inch HMI, V3.0**, with an **ESP32-S3-WROOM-1-N4R8**, an **800×480 RGB display**, **8 MB PSRAM**, and **4 MB flash**.

## Current Status

**Gameplay is working on real ESP32-S3 hardware: a full campaign run from Mission 0 to the end has been completed, and demo playback runs to completion.**

*Last updated: September 27, 2026. Experimental development build.*

**Main menu, item by item:**

- [x] New Game confirmed; a full campaign has been played from Mission 0 to the end.
- [ ] Load Game... not yet confirmed until Save Game is implemented. Save data is being handled with a custom SafeSaveData system because the user can initiate a reset at any time.
- [ ] Multiplayer... reaches one level in (Start a Network Game / Join a Network Game / Modem-Serial Game) and exits cleanly, but none of those options do anything yet. Multiplayer backend candidates are being considered, including ESP-NOW.
- [x] Options...
- [x] Change Pilots...
- [x] View Demo...
- [x] High Scores
- [ ] Benchmark...
- [x] Credits
- [x] Quit
  - [x] Load Level... implemented via the on-screen keyboard. BLE keyboard support is being considered.
  - [ ] Play Song... not yet viable without a music backend.

dESPcent now runs the original engine through startup, menus, mission briefings, and into working gameplay. Demo playback also runs to completion. Current work is focused on two things: systematically checking the remaining screens and adapting their display updates and wait loops to the ESP32 graphics backend and FreeRTOS scheduler, and optimizing performance.

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
- An on-screen keyboard (gamepad grid + GT911 tap) for pilot-callsign and level-number entry, confirmed working on hardware.
- Change Pilots, confirmed working on hardware.

### Current Tasks

Some original DOS screen loops still need explicit display updates and scheduler yields of their own. On this port, drawing into the indexed screen buffer must be followed by `gr_present()` to update the panel, and polling/timing loops need `vTaskDelay(...)` so other FreeRTOS tasks can run. 

Working gameplay, completed demos, and now hundreds-of-frames-stable runs are milestones, not a claim that every screen, mission, or feature has been validated.

## Screens & Menus

### Startup

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

### Hardware setup & calibration

<p align="center">
  <img width="900" alt="dESPcent DOS-style hardware setup screen: system info page" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_setup_system_info.jpg?raw=true" />
</p>

<p align="center">
  <em>The DOS-style hardware setup screen's System Info page, reporting board and firmware identity from within the original ASCII-style setup program.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent DOS-style hardware setup screen: data page" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_setup_data.jpg?raw=true" />
</p>

<p align="center">
  <em>The setup program's Data page, covering game-data paths and the archives detected on the SD card.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent DOS-style hardware setup screen: controller page" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_setup_controller.jpg?raw=true" />
</p>

<p align="center">
  <em>The setup program's Controller page, confirming the BLE gamepad binding before entering the game.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent joystick calibration screen: left stick deadzone" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_setup_left_stick_dz.jpg?raw=true" />
</p>

<p align="center">
  <em>Descent's original joystick-calibration path, adapted to walk the BLE gamepad's left stick through its deadzone.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent joystick calibration screen: right stick deadzone" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_setup_right_stick_dz.jpg?raw=true" />
</p>

<p align="center">
  <em>The same calibration flow for the right stick.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent joystick calibration screen: trigger deadzone" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_setup_trigger_dz.jpg?raw=true" />
</p>

<p align="center">
  <em>Trigger deadzone calibration, completing the calibration sequence on the BLE pad.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent DOS-style hardware setup screen: exit page" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_setup_exit.jpg?raw=true" />
</p>

<p align="center">
  <em>Exiting the setup program and handing control back to the game.</em>
</p>

### Pilot, menus, and briefings

<p align="center">
  <img width="900" alt="dESPcent Select Pilot screen listing saved .PLR files" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_pilot_select.jpg?raw=true" />
</p>

<p align="center">
  <em>The Select Pilot screen, listing saved <code>.PLR</code> profiles read directly from the SD card.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent pilot callsign entry using the new on-screen keyboard" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_pilot_name_entry.jpg?raw=true" />
</p>

<p align="center">
  <em><strong>New:</strong> typing a pilot callsign with the on-screen keyboard. The alphanumeric grid, navigable by gamepad or a GT911 tap, fills the one gap a DOS-only input model left on hardware with no physical keyboard.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent main menu on ESP32-S3 hardware" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_main_menu.jpg?raw=true" />
</p>

<p align="center">
  <em>The original Descent main menu, fully visible and controller-navigable on the CrowPanel.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent New Game mission-select screen" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_new_game_mission_select.jpg?raw=true" />
</p>

<p align="center">
  <em>New Game's mission-select list.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent New Game difficulty-select screen" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_new_game_difficulty_select.jpg?raw=true" />
</p>

<p align="center">
  <em>Difficulty selection, immediately before the mission briefing begins.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent Options menu on ESP32-S3 hardware" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_options_menu.jpg?raw=true" />
</p>

<p align="center">
  <em>The Options menu, including the Sound FX volume slider verified end-to-end through the ESP32 I²S audio path.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent View Demo file list" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_view_demo_menu.jpg?raw=true" />
</p>

<p align="center">
  <em>The View Demo file list.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent High Scores screen" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_high_scores.jpg?raw=true" />
</p>

<p align="center">
  <em>The High Scores table.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent Credits screen" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_credits.jpg?raw=true" />
</p>

<p align="center">
  <em>The credits screen.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent Quit confirmation dialog" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_quit_confirm.jpg?raw=true" />
</p>

<p align="center">
  <em>The Quit confirmation dialog.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent mission briefing on ESP32-S3 hardware" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_mission_briefing.jpg?raw=true" />
</p>

<p align="center">
  <em>The mission briefing system running on hardware, including original backgrounds, palette effects, timed text, and page flow.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent character briefing screen on ESP32-S3 hardware" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_character_briefing.jpg?raw=true" />
</p>

<p align="center">
  <em>A later briefing page with character artwork and scrolling mission text. The original 320×200 presentation is scaled 2× to 640×400 and centered on the 800×480 panel.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent numeric on-screen keyboard entering a Load Level mission number" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_load_level_keyboard.jpg?raw=true" />
</p>

<p align="center">
  <em><strong>New:</strong> the same on-screen keyboard module in its numeric-only layout, entering a mission number for Load Level.</em>
</p>

### Gameplay

<p align="center">
  <img width="900" alt="dESPcent in-game cockpit frame on ESP32-S3 hardware" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_gameplay_cockpit.jpg?raw=true" />
</p>

<p align="center">
  <em>In-game cockpit and HUD during working gameplay. Player and robot physics active, original software renderer producing the scene natively.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent debug/performance overlay during gameplay" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_debug_overlay.jpg?raw=true" />
</p>

<p align="center">
  <em><strong>New:</strong> the retuned debug/performance overlay. Shorter lines render faster, still covering frame timing, object counts, and memory.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent demo playback running to completion" src="https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_demo_playback.jpg?raw=true" />
</p>

<p align="center">
  <em>Demo playback running to completion in the cockpit view.</em>
</p>

<details>
<summary><strong>Archived screenshots (superseded by the gallery above)</strong></summary>

These were the working images before this pass; several point at screens the gallery above now shows with a dedicated, on-disk photo instead (setup/calibration pages, pilot select, main menu, briefings, cockpit). Kept for reference rather than deleted.

<p align="center">
  <img width="900" alt="dESPcent DOS-style hardware setup screen (archived)" src="https://github.com/user-attachments/assets/425561aa-bea6-4958-85b6-7b98dcaeb72d?raw=true" />
</p>

<p align="center">
  <em>A DOS-style hardware setup screen inspired by the original Descent ASCII setup program.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent pilot-name screen on ESP32-S3 hardware (archived)" src="https://github.com/user-attachments/assets/e19f119b-408c-474e-84ac-e4683a3767ea" />
</p>

<p align="center">
  <em>The original pilot-name flow running natively on the ESP32-S3. Existing configuration data is already being read by the engine.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent joystick calibration screen on ESP32-S3 hardware (archived)" src="https://github.com/user-attachments/assets/4404d314-385b-4ad4-b716-1e80a82e5c5e" />
</p>

<p align="center">
  <em>Descent's original joystick-calibration path, reached through the ported controller/input layer.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent main menu on ESP32-S3 hardware (archived)" src="https://github.com/user-attachments/assets/a7188166-6f86-40c6-bd52-1e05daa156ea" />
</p>

<p align="center">
  <em>The original Descent main menu, fully visible and controller-navigable on the CrowPanel.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent mission briefing on ESP32-S3 hardware (archived)" src="https://github.com/user-attachments/assets/e9dd06a0-48b5-4424-abcb-f401a3160feb" />
</p>

<p align="center">
  <em>The mission briefing system running on hardware, including original backgrounds, palette effects, timed text, and page flow.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent character briefing screen on ESP32-S3 hardware (archived)" src="https://github.com/user-attachments/assets/02d75b06-1713-40d5-bdb0-3d621c8f3e4f" />
</p>

<p align="center">
  <em>A later briefing page with character artwork and scrolling mission text. The original 320×200 presentation is scaled 2× to 640×400 and centered on the 800×480 panel.</em>
</p>

<p align="center">
  <img width="900" alt="dESPcent first in-game cockpit frame on ESP32-S3 hardware (archived)" src="https://github.com/user-attachments/assets/f1430721-5c5c-406a-8a5a-5e5b68b52a34" />
</p>

<p align="center">
  <em>First in-game cockpit frame on the ESP32-S3. The game loop is active, player and robot physics are running, and the original software renderer is producing the scene natively.</em>
</p>

</details>

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
| **SD** - MOSI / MISO / SCLK / CS | 11 / 13 / 12 / 10 |
| **I²C** - SDA / SCL | 19 / 20 |
| **I²S** - DOUT / LRCLK / BCLK | 17 / 18 / 42 |
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

BLE controller support is confirmed on hardware, in full: a complete campaign run, Mission 0 to the end, was flown entirely with the Xbox Wireless Controller over BLE.

The Xbox controller path is integrated into Descent's joystick/menu contracts and handles both menu navigation and in-flight controls:

- **A / Start** → Enter/confirm in menu-style interfaces.
- **B** → Escape/back.
- **D-pad** → menu navigation.

In gameplay, the right stick controls yaw/pitch, the left stick controls horizontal/vertical slide, RT/LT give proportional forward/reverse thrust, and RB/LB bank. A/B fire primary/secondary weapons, X launches a flare, Y cycles primary weapons, L3 holds afterburner-style boost thrust, and R3 drops a proximity bomb. D-pad cycles weapons, Start pauses, View/Back opens board setup, and Share prints a diagnostic dump to serial and toggles the debug overlay.

The GT911 touch-backed mouse implementation is also confirmed working: a tap registers as a left mouse click anywhere the engine expects one, including dismissing menus and driving the on-screen keyboard.

## Audio

Digital sound is working on hardware.

The original Descent sound tables and sample data load successfully. Current builds report 108 sound slots in use and roughly 404 KB of actual sample payload selected from the larger sound reservation.

dESPcent replaces the original DOS sound-hardware backend with an ESP32 software mixer and I²S output path. Mixed audio is sent through the CrowPanel's NS4168 amplifier to external speakers.

Sound effects are active during normal gameplay, and the original Sound FX volume control works end-to-end, including its menu test sound and volume scaling.

Descent also exposes a Sound Channels control in the Custom Detail Level menu. This provides an original engine mechanism for varying the number of simultaneous digital sound channels, making it useful both as a gameplay setting and as a measurable CPU-load control on the ESP32-S3.

### Music

Music is not implemented yet.

This is a separate problem from digital sound effects. The original game provides sequenced music data rather than ready-to-play PCM audio, so dESPcent needs both a sequence player and a synthesizer or another method of producing the final audio stream.

Several approaches are viable, with different costs:

- **Software FM/OPL synthesis** would preserve the character of period DOS hardware and avoid large instrument banks, but adds continuous CPU load to an ESP32-S3 already performing software rendering, game simulation, sound mixing, Bluetooth, and RGB display output.

- **General MIDI software synthesis** could provide music closer to a wavetable/MIDI setup, but requires an instrument bank in addition to synthesizer code. That consumes storage and potentially substantial RAM, while polyphonic synthesis adds another ongoing CPU workload.

- **Pre-rendered music** could convert the original tracks to PCM or a compressed streaming format ahead of time and play them from SD. This is computationally much cheaper than real-time synthesis, but increases storage requirements and would no longer be synthesizing the original sequence data at runtime.

- **External synthesis hardware** is also possible, but would add hardware requirements and is not appropriate as the baseline dESPcent configuration.

- **No music** remains a valid configuration. Digital sound effects and gameplay do not depend on the music backend.

The eventual choice will be based on measured cost rather than assuming that real-time synthesis is affordable. dESPcent already has several significant continuous workloads, so music must coexist with rendering, simulation, digital sound mixing, BLE input, and display scanout without compromising gameplay.

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

- Root-causing the remaining off-by-one write into the offscreen canvas. The corruption is now localized to a specific 1-byte, per-frame signature; the exact write site is not yet traced.
- Remaining screen loops that need explicit `gr_present()` calls or FreeRTOS scheduler yields.
- Final in-game controller axis behavior and bindings.
- Full audio playback.
- Performance characterization under real gameplay load: multiple robots, projectiles, collision checks, AI, sound mixing, and sustained rendering.
- Network support. The main-menu Multiplayer entry is reachable and one level deep. It shows Start a Network Game / Join a Network Game / Modem-Serial Game and exits cleanly back to the main menu, but none of those options do anything past that point yet.
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
