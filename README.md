# dESPcent

**dESPcent** is an experimental native port of the original **Descent** engine to the **ESP32-S3**.

The goal is simple: find out how much mid-1990s PC game an ESP32-S3 can run when it executes the engine directly, without emulating the PC underneath it.

The current target is an **Elecrow CrowPanel 7-inch HMI, V3.0**, with an **ESP32-S3-WROOM-1-N4R8**, an **800×480 RGB display**, **8 MB PSRAM**, and **4 MB flash**.

## Current Status

**It's alive: native startup is running on hardware. Gameplay is still a work in progress.**

The supplied hardware log confirms that dESPcent:

- Boots with ESP-IDF v6.0.1 at 240 MHz and detects 8 MB PSRAM and 4 MB flash.
- Mounts the SD card and starts the RGB display.
- Authenticates and connects to an Xbox Wireless Controller over BLE.
- Passes the required `DESCENT.HOG` and `DESCENT.PIG` readability checks and identifies the known registered 1.0 reference data pair.
- Reads battery charge and voltage from the LC709203F fuel gauge.
- Enters Descent's native `INFERNO` startup and prints the registered v1.5 engine banner.

![dESPcent startup POST on the CrowPanel, showing mounted SD storage and passing Descent archive checks.](https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_post_screen.jpg?raw=true)

*Working POST on hardware. The memory figures in this photograph belong to that build and startup stage; they are not the remaining game heap in every configuration.*

```
I (25) boot: ESP-IDF v6.0.1 2nd stage bootloader
I (25) boot: compile time Sep 19 2026 23:42:51
I (25) boot: Multicore bootloader
I (25) boot: chip revision: v0.2
I (28) boot: efuse block revision: v1.3
I (31) qio_mode: Enabling default flash chip QIO
I (36) boot.esp32s3: Boot SPI Speed : 80MHz
I (40) boot.esp32s3: SPI Mode       : QIO
I (43) boot.esp32s3: SPI Flash Size : 4MB
I (47) boot: Enabling RNG early entropy source...
I (52) boot: Partition Table:
I (54) boot: ## Label            Usage          Type ST Offset   Length
I (61) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (67) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (74) boot:  2 factory          factory app      00 00 00010000 003f0000
I (80) boot: End of partition table
I (84) esp_image: segment 0: paddr=00010020 vaddr=3c0e0020 size=30724h (198436) map
I (121) esp_image: segment 1: paddr=0004074c vaddr=3fc9f400 size=080c8h ( 32968) load
I (127) esp_image: segment 2: paddr=0004881c vaddr=40378000 size=077fch ( 30716) load
I (133) esp_image: segment 3: paddr=00050020 vaddr=42000020 size=d50fch (872700) map
I (266) esp_image: segment 4: paddr=00125124 vaddr=4037f7fc size=0fb2ch ( 64300) load
I (278) esp_image: segment 5: paddr=00134c58 vaddr=50000000 size=00024h (    36) load
I (289) boot: Loaded app from partition at offset 0x10000
I (289) boot: Disabling RNG early entropy source...
I (300) octal_psram: vendor id    : 0x0d (AP)
I (300) octal_psram: dev id       : 0x02 (generation 3)
I (300) octal_psram: density      : 0x03 (64 Mbit)
I (302) octal_psram: good-die     : 0x01 (Pass)
I (307) octal_psram: Latency      : 0x01 (Fixed)
I (311) octal_psram: VCC          : 0x01 (3V)
I (315) octal_psram: SRF          : 0x01 (Fast Refresh)
I (320) octal_psram: BurstType    : 0x01 (Hybrid Wrap)
I (325) octal_psram: BurstLen     : 0x01 (32 Byte)
I (329) octal_psram: Readlatency  : 0x02 (10 cycles@Fixed)
I (335) octal_psram: DriveStrength: 0x00 (1/1)
I (339) MSPI Timing: Enter psram timing tuning
I (344) esp_psram: Found 8MB PSRAM device
I (347) esp_psram: Speed: 80MHz
I (365) mmu_psram: Read only data copied and mapped to SPIRAM
I (426) mmu_psram: Instructions copied and mapped to SPIRAM
I (426) cpu_start: Multicore app
I (671) esp_psram: SPI SRAM memory test OK
I (708) cpu_start: GPIO 44 and 43 are used as console UART I/O pins
I (708) cpu_start: Pro cpu start user code
I (708) cpu_start: cpu freq: 240000000 Hz
I (710) app_init: Application information:
I (714) app_init: Project name:     dESPcent
I (718) app_init: App version:      1
I (721) app_init: Compile time:     Sep 20 2026 00:49:03
I (726) app_init: ELF file SHA256:  be33e02d6...
I (730) app_init: ESP-IDF:          v6.0.1
I (734) efuse_init: Min chip rev:     v0.0
I (738) efuse_init: Max chip rev:     v0.99 
I (742) efuse_init: Chip rev:         v0.2
I (746) heap_init: Initializing. RAM available for dynamic allocation:
I (752) heap_init: At 3FCD04E0 len 00019230 (100 KiB): RAM
I (757) heap_init: At 3FCE9710 len 00005724 (21 KiB): RAM
I (762) heap_init: At 600FE000 len 00001FE8 (7 KiB): RTCRAM
I (768) esp_psram: Adding pool of 5788K of PSRAM memory to heap allocator
I (774) esp_psram: Adding pool of 62K of PSRAM memory gap generated due to end address alignment of drom to the heap allocator
I (786) spi_flash: detected chip: generic
I (789) spi_flash: flash io: qio
I (792) sleep_gpio: Configure to isolate all GPIO pins in sleep state
I (798) sleep_gpio: Enable automatic switching of GPIO sleep configuration
I (805) coexist: coex firmware version: b00e8cb
I (809) coexist: coexist rom version e7ae62f
I (813) main_task: Started on CPU0
I (823) esp_psram: Reserving pool of 32K of internal memory for DMA/internal allocations
I (823) main_task: Calling app_main()
I (823) descent-board: ESP-IDF v6.0.1
I (833) descent-board: ESP32-S3 cores=2 revision=2
I (833) descent-board: heap=6069028 bytes, PSRAM heap=5987804 bytes
I (843) descent-board: RGB panel=800x480 backlight GPIO=2
I (843) descent-board: I2C SDA/SCL=19/20 PCA9557=0x18
I (853) descent-board: SD MOSI/MISO/SCLK/CS=11/13/12/10
I (853) descent-board: I2S LRCLK/BCLK/DOUT=18/42/17
I (983) sdspi_transaction: cmd=52, R1 response: command not supported
I (1023) sdspi_transaction: cmd=5, R1 response: command not supported
I (1033) descent-board: SD card mounted at /sdcard (SD16G, 15360 MB)
I (1043) descent-board: CrowPanel V3.0 board startup complete (sd_mounted=1)
I (1043) descent: Heap after board init: total=6091615 internal=110663 DMA=102875 PSRAM=5980952 largest_internal=51200 largest_DMA=51200 largest_PSRAM=5898240
I (1053) descent: Heap before display reserve: total=6091615 internal=110663 DMA=102875 PSRAM=5980952 largest_internal=51200 largest_DMA=51200 largest_PSRAM=5898240
I (1103) descent-display: RGB buffers reserved (scanout stopped): 800x480 @ 15000000 Hz pclk, fb0=0x3c259740 fb1=0x3c314f80
I (1103) descent: Heap after display reserve: total=4522123 internal=77179 DMA=69391 PSRAM=4444944 largest_internal=34816 largest_DMA=34816 largest_PSRAM=4325376
I (1113) BLE_INIT: BT controller compile version [b7de11e]
I (1123) BLE_INIT: Using main XTAL as clock source
I (1123) BLE_INIT: Feature Config, ADV:1, BLE_50:1, DTM:1, SCAN:1, CCA:0, SMP:1, CONNECT:1
I (1133) BLE_INIT: Bluetooth MAC: 00:00:00:00:00:00
I (1133) phy_init: phy_version 711,97bcf0a2,Aug 25 2025,19:04:10
I (1183) descent: Heap after BLE begin: total=4459623 internal=18607 DMA=10819 PSRAM=4441016 largest_internal=10752 largest_DMA=10752 largest_PSRAM=4325376
I (1193) descent: BLE controller ready; scanning for Xbox controller
I (1203) descent: Heap after BLE init: total=4453283 internal=12319 DMA=4531 PSRAM=4440964 largest_internal=7680 largest_DMA=4352 largest_PSRAM=4325376
I (1213) descent: Heap before display init: total=4453283 internal=12319 DMA=4531 PSRAM=4440964 largest_internal=7680 largest_DMA=4352 largest_PSRAM=4325376
I (1243) descent-display: RGB scanout started
I (1243) descent: Heap after display init: total=4453283 internal=12319 DMA=4531 PSRAM=4440964 largest_internal=7680 largest_DMA=4352 largest_PSRAM=4325376
I (1253) descent: Heap before POST: total=4453283 internal=12319 DMA=4531 PSRAM=4440964 largest_internal=7680 largest_DMA=4352 largest_PSRAM=4325376
BLE HID: scan found 1 device(s), opening match 'Xbox Wireless Controller'
W (6443) BT_HCI: hcif disc complete: hdl 0x1, rsn 0x3e dev_find 1
W (6643) BT_HCI: hcif disc complete: hdl 0x1, rsn 0x3e dev_find 1
I (8673) ESP_HID_GAP: BLE GAP AUTH SUCCESS
BLE HID: connected to 'Xbox Wireless Controller'
I (20553) descent-cdrom: Descent CD image found: /sdcard/descent.iso
I (20553) descent-post: DESCENT.HOG: PASS
I (20553) descent-post: DESCENT.PIG: PASS
I (20563) descent-post: DATA: REGISTERED 1.0 REF; PIG: EARLY D1 LAYOUT
I (20593) descent: Heap after POST: total=4441451 internal=4215 DMA=3171 PSRAM=4437236 largest_internal=2560 largest_DMA=2560 largest_PSRAM=4325376
I (20603) descent: Heap before INFERNO: total=4443499 internal=6263 DMA=3171 PSRAM=4437236 largest_internal=2560 largest_DMA=2560 largest_PSRAM=4325376
I (20773) descent: LC709203F found, starting 10-second poll
I (20773) descent: Entering INFERNO with data directory /sdcard/DESCENT/
I (20783) descent: Batt: 95.4% (4.14V)
Text: 555 original entries + 66 later-version compatibility entries

DESCENT   Registered v1.5 Jan 5, 1996
Copyright (C) 1994, 1995 Parallax Software Corporation
DESCENT is a trademark of Interplay Productions, Inc.

Type 'DESCENT -help' for a list of command-line options.
Available memory
Internal free: 6247 bytes; largest block: 2560 bytes
PSRAM detected: 8388608 bytes; free: 4421304 bytes; largest block: 4325376 bytes
Flash detected: 4194304 bytes; configured: 4MB
Insufficient available memory: original 7.5 MiB startup requirement not met.
I (21023) descent: INFERNO returned 1
I (20873) main_task: Returned from app_main()
```

### Current startup blocker

The supplied log ends with:

```text
Internal free: 6247 bytes; largest block: 2560 bytes
PSRAM detected: 8388608 bytes; free: 4421304 bytes; largest block: 4325376 bytes
Flash detected: 4194304 bytes; configured: 4MB
Insufficient available memory: original 7.5 MiB startup requirement not met.
I (21023) descent: INFERNO returned 1
```

This is a deliberate startup rejection, not a crash. The current memory check retains a 7.5 MiB free-memory threshold. By this stage, engine data, code, display buffers, Bluetooth, and other services already occupy part of the available RAM.

The next step is to establish the engine's actual allocation requirements and adapt the check accordingly, while accounting for the very limited internal heap. Passing or bypassing that check alone would not establish that the game fits or runs correctly.

Opening logos, menus, level rendering, playable controls, sound, and music are not demonstrated by this startup log. There is no measured gameplay frame rate yet.

## Goals

- Run the original Descent engine natively on ESP32-S3.
- Preserve its fixed-point mathematics, software renderer, and gameplay behavior where practical.
- Replace DOS-specific hardware access with ESP-IDF implementations.
- Use internal RAM and PSRAM according to measured requirements.
- Load game data from SD storage.
- Support gamepad controls, sound, and music.
- Render at a practical internal resolution and scale to the 800×480 panel.
- Keep the result recognizable as a port of Descent, rather than a ground-up reimplementation.

## Target Hardware

| Component | Current target or prototype |
| --- | --- |
| Board | Elecrow CrowPanel 7-inch HMI, V3.0 |
| Processor | Dual-core ESP32-S3, configured at 240 MHz |
| Memory | Internal SRAM plus 8 MB external octal PSRAM, configured at 80 MHz |
| Flash | 4 MB, configured for QIO at 80 MHz |
| Storage | FAT-formatted SD card over SPI; the supplied boot log reports a nominal 16 GB card |
| Display | 800×480 RGB panel, RGB565 output |
| Touch | GT911 capacitive controller; V3.0 startup uses the PCA9557 I²C expander |
| Audio hardware | NS4168 amplifier with an I²S digital interface; engine audio output remains unfinished |
| Wireless controller | BLE Xbox Wireless Controller; decoder targets model 1914 |
| Prototype battery | MakerFocus 3.7 V, 3,700 mAh battery |
| Battery gauge | Adafruit LC709203F breakout over I²C |

![Rear of the prototype, showing the MakerFocus 3700 mAh battery and Adafruit LC709203F fuel gauge.](https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_proto_0.1_rear.jpg?raw=true)

The battery and gauge are additions to this prototype. Their presence does not establish battery runtime, which has not been measured here. A 32 GB SD card is not a project requirement.

### Peripheral connections

| Interface | GPIO assignment |
| --- | --- |
| SD MOSI / MISO / SCLK / CS | 11 / 13 / 12 / 10 |
| I²C SDA / SCL | 19 / 20 |
| I²S data out / LRCLK / BCLK | 17 / 18 / 42 |
| LCD backlight | 2 |

Other ESP32-S3 boards may be supported later, but this is currently a board-specific port.

## Memory and Display Strategy

Having 8 MB of PSRAM does not mean that 8 MB is available to the game. Large engine arrays, firmware instructions and read-only data, display buffers, and eligible Bluetooth allocations share that external memory. Internal RAM is still needed for resources such as task stacks and display DMA buffers.

The current configuration enables execution from PSRAM: ESP-IDF copies flash-backed instructions and read-only data into PSRAM at startup. This consumes additional PSRAM and supports the RGB bounce-buffer path during later flash operations, such as saving Bluetooth pairing information. The firmware remains stored in flash.

The display driver currently reserves:

| Buffer | Size |
| --- | ---: |
| One 800×480 RGB565 framebuffer | 768,000 bytes |
| Two RGB565 framebuffers | 1,536,000 bytes |
| Two internal DMA bounce buffers, 10 lines each | 32,000 bytes total |
| Current 800×480 indexed engine buffer, allocated by graphics initialization | 384,000 bytes |

Display buffers are reserved before Bluetooth initialization to secure the required internal DMA allocations. RGB scanout starts afterward. Engine graphics initialization reuses the existing panel.

At the configured 15 MHz pixel clock and 928×525 total timing, the calculated panel refresh rate is approximately **30.79 Hz**. Active RGB565 pixel payload is approximately **23.65 MB/s**, before rendering writes and other memory traffic. This is a display timing calculation, not a measured game frame rate.

### Planned lower-resolution rendering

The current graphics implementation uses an 800×480 indexed backing buffer. The proposed **384×240** engine framebuffer and **2× scaling to 768×480**, with **16-pixel side borders**, are not yet implemented.

A 384×240 indexed buffer would require 92,160 bytes. It would reduce the number of engine pixels to render, but it would not by itself remove the full-size RGB output buffers or the panel's continuous scanout bandwidth.

## Controls and Audio

BLE controller connection has been demonstrated. A joystick adapter is present in the source, but axis orientation, calibration, bindings, and in-game behavior still need hardware validation.

The GT911 has a touch-backed mouse implementation. That does not yet establish touch-selectable game menus; menu integration and hardware validation remain separate work.

The board's I²S audio pins are identified, but the current sound code does not provide a working ESP32 I²S playback backend. Sound effects and music remain goals.

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

### Optional ISO installation

Instead of copying the two archives yourself, place one supported ISO image in the SD root or `/DESCENT`. If a readable loose pair is already available, it is used without extraction. Otherwise, the installer looks for both archives together inside the image, stages and verifies them, and promotes the result to `/DESCENT_AUTO`.

Existing game-data directories are not overwritten. The ISO can be removed after successful installation; runtime uses the extracted archives. Compressed installers and raw BIN/CUE images are not supported by this reader.

## Development Philosophy

Accuracy and functionality come first. Preserve the original engine where practical, establish a working baseline, and optimize according to measurements of CPU use, memory, rendering, and I/O.

If something is slow, we want to know **why**. If something changes for the ESP32-S3, there should be a concrete reason for the change.

This is an ESP-IDF CMake project. The supplied hardware run used **ESP-IDF v6.0.1** with the `esp32s3` target. The root `sdkconfig`, `sdkconfig.defaults`, and `partitions.csv` describe the current configuration. The custom partition table provides a single factory application partition within 4 MB flash; it does not provide OTA slots.

The files under [docs](docs) record individual porting steps and checks. Some describe earlier snapshots and should not be read as current feature status.

## Source and Licensing

The source baseline is the original **Descent 1 v1.5** release, rather than D1X or DXX-Rebirth. Platform-specific code is being adapted or replaced for ESP32-S3 while retaining the original engine structure and notices.

The ported Parallax source files carry a license notice restricting use to non-commercial, royalty- or revenue-free purposes. The project should not be described as having a blanket permissive open-source license. Applicable source and third-party notices must be retained; consolidated distribution licensing documentation remains to be completed.

Game data is separate from the source release and is not made freely distributable by the availability of engine source.

**Descent** is a trademark of its respective owners. dESPcent is an independent project and is not affiliated with or endorsed by those owners.

## Why?

Because an ESP32-S3 running Descent would be ridiculous.

And now the startup code is running. Next comes the game.
