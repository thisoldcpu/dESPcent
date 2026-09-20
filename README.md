# dESPcent

**dESPcent** is an experimental native port of the original **Descent** engine to the **ESP32-S3**.

The goal is simple: find out how much mid-1990s PC game an ESP32-S3 can run when it executes the engine directly, without emulating the PC underneath it.

The current target is an **Elecrow CrowPanel 7-inch HMI, V3.0**, with an **ESP32-S3-WROOM-1-N4R8**, an **800×480 RGB display**, **8 MB PSRAM**, and **4 MB flash**.

## Current Status

**It's alive: native startup is running on hardware. Gameplay is still a work in progress.**

The supplied hardware log confirms that dESPcent:

- Boots with ESP-IDF v6.0.1 at 240 MHz and detects 8 MB PSRAM and 4 MB flash.
- Mounts the SD card and starts the 800×480 RGB display.
- Authenticates and connects to an Xbox Wireless Controller over BLE.
- Runs an animated hardware/data POST with SD verification, battery status, and touch-to-continue handoff.
- Passes the required `DESCENT.HOG` and `DESCENT.PIG` readability checks and identifies the known registered 1.0 reference data pair.
- Enters Descent's native `INFERNO` startup and prints the registered v1.5 engine banner.
- Initializes the original palette and font systems.
- Successfully decodes and renders the Interplay, Parallax Software, and Descent startup screens directly to the ESP32-S3's RGB panel.

![dESPcent startup POST verifying Descent data on the CrowPanel.](https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_post_verifying.jpg?raw=true)
*The animated POST verifies SD data while hardware status remains visible, including live battery state.*

![dESPcent startup POST complete and ready to launch.](https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_post_verifed.jpg?raw=true)
*Verification complete and ready to launch. Touch input hands control off to the original Descent startup sequence.*

![Interplay logo rendered on the CrowPanel.](https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_interplay_logo.jpg?raw=true)
*The software renderer outputs the Interplay logo directly to the ESP32-S3's 800×480 RGB panel.*

![Parallax Software logo rendered on the CrowPanel.](https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_parallax_logo.jpg?raw=true)
*The Parallax Software logo sequence rendered on the CrowPanel.*

![Descent logo rendered on the CrowPanel.](https://github.com/thisoldcpu/dESPcent/blob/main/images/despcent_descent_logo.jpg?raw=true)
*The original Descent logo on the ESP32-S3 as the native startup sequence continues.*

```
PS C:\Projects\dESPcent> $env:IDF_PATH = 'C:\esp\v6.0.1\esp-idf';
PS C:\Projects\dESPcent>  & 'C:\Espressif\tools\python\v6.0.1\venv\Scripts\python.exe' 'C:\esp\v6.0.1\esp-idf\tools\idf_monitor.py' -p COM3 -b 115200 --toolchain-prefix xtensa-esp32s3-elf- --make '''C:\Espressif\tools\python\v6.0.1\venv\Scripts\python.exe'' ''C:\esp\v6.0.1\esp-idf\tools\idf.py''' --target esp32s3 'c:\Projects\dESPcent\build\dESPcent.elf'
--- Warning: GDB cannot open serial ports accessed as COMx
--- Using \\.\COM3 instead...
--- esp-idf-monitor 1.9.0 on \\.\COM3 115200
--- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H
ESP-ROM:esp32s3-20210327
Build:Mar 27 2021
rst:0x1 (POWERON),boot:0x8 (SPI_FAST_FLASH_BOOT)
SPIWP:0xee
mode:DIO, clock div:1
load:0x3fce2820,len:0x1664
load:0x403c8700,len:0xf38
load:0x403cb700,len:0x31ec
entry 0x403c8930
I (25) boot: ESP-IDF v6.0.1 2nd stage bootloader
I (25) boot: compile time Sep 20 2026 05:04:37
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
I (84) esp_image: segment 0: paddr=00010020 vaddr=3c0e0020 size=306a4h (198308) map
I (121) esp_image: segment 1: paddr=000406cc vaddr=3fc9f400 size=080c8h ( 32968) load
I (127) esp_image: segment 2: paddr=0004879c vaddr=40378000 size=0787ch ( 30844) load
I (133) esp_image: segment 3: paddr=00050020 vaddr=42000020 size=d54a4h (873636) map
I (266) esp_image: segment 4: paddr=001254cc vaddr=4037f87c size=0faach ( 64172) load
I (279) esp_image: segment 5: paddr=00134f80 vaddr=50000000 size=00024h (    36) load
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
I (721) app_init: Compile time:     Sep 20 2026 05:04:34
I (726) app_init: ELF file SHA256:  273036250...
I (731) app_init: ESP-IDF:          v6.0.1
I (734) efuse_init: Min chip rev:     v0.0
I (738) efuse_init: Max chip rev:     v0.99 
I (742) efuse_init: Chip rev:         v0.2
I (746) heap_init: Initializing. RAM available for dynamic allocation:
I (752) heap_init: At 3FCD04E0 len 00019230 (100 KiB): RAM
I (757) heap_init: At 3FCE9710 len 00005724 (21 KiB): RAM
I (763) heap_init: At 600FE000 len 00001FE8 (7 KiB): RTCRAM
I (768) esp_psram: Adding pool of 5788K of PSRAM memory to heap allocator
I (774) esp_psram: Adding pool of 62K of PSRAM memory gap generated due to end address alignment of drom to the heap allocator
I (786) spi_flash: detected chip: generic
I (789) spi_flash: flash io: qio
I (792) sleep_gpio: Configure to isolate all GPIO pins in sleep state
I (799) sleep_gpio: Enable automatic switching of GPIO sleep configuration
I (805) coexist: coex firmware version: b00e8cb
I (809) coexist: coexist rom version e7ae62f
I (814) main_task: Started on CPU0
I (824) esp_psram: Reserving pool of 32K of internal memory for DMA/internal allocations
I (824) main_task: Calling app_main()
I (824) descent-board: ESP-IDF v6.0.1
I (834) descent-board: ESP32-S3 cores=2 revision=2
I (834) descent-board: heap=6064548 bytes, PSRAM heap=5987932 bytes
I (844) descent-board: RGB panel=800x480 backlight GPIO=2
I (844) descent-board: I2C SDA/SCL=19/20 PCA9557=0x18
I (854) descent-board: SD MOSI/MISO/SCLK/CS=11/13/12/10
I (854) descent-board: I2S LRCLK/BCLK/DOUT=18/42/17
I (984) sdspi_transaction: cmd=52, R1 response: command not supported
I (1024) sdspi_transaction: cmd=5, R1 response: command not supported
I (1034) descent-board: SD card mounted at /sdcard (SD16G, 15360 MB)
I (1044) descent-board: CrowPanel V3.0 board startup complete (sd_mounted=1)
I (1044) descent: Heap after board init: total=6087135 internal=106055 DMA=98267 PSRAM=5981080 largest_internal=47104 largest_DMA=47104 largest_PSRAM=5898240
I (1054) descent: Heap before display reserve: total=6087135 internal=106055 DMA=98267 PSRAM=5981080 largest_internal=47104 largest_DMA=47104 largest_PSRAM=5898240
I (1104) descent-display: RGB buffers reserved (scanout stopped): 800x480 @ 15000000 Hz pclk, fb0=0x3c259740 fb1=0x3c314f80
I (1104) descent: Heap after display reserve: total=4517643 internal=72571 DMA=64783 PSRAM=4445072 largest_internal=31744 largest_DMA=31744 largest_PSRAM=4325376
I (1114) BLE_INIT: BT controller compile version [b7de11e]
I (1124) BLE_INIT: Using main XTAL as clock source
I (1124) BLE_INIT: Feature Config, ADV:1, BLE_50:1, DTM:1, SCAN:1, CCA:0, SMP:1, CONNECT:1
I (1134) BLE_INIT: Bluetooth MAC: 74:4d:bd:9d:91:b6
I (1134) phy_init: phy_version 711,97bcf0a2,Aug 25 2025,19:04:10
I (1194) descent: Heap after BLE begin: total=4457199 internal=16055 DMA=8347 PSRAM=4441144 largest_internal=8192 largest_DMA=8192 largest_PSRAM=4325376
I (1194) descent: BLE controller ready; scanning for Xbox controller
I (1204) descent: Heap after BLE init: total=4453931 internal=12839 DMA=5131 PSRAM=4441092 largest_internal=7680 largest_DMA=4864 largest_PSRAM=4325376
I (1214) descent: Heap before display init: total=4453931 internal=12839 DMA=5131 PSRAM=4441092 largest_internal=7680 largest_DMA=4864 largest_PSRAM=4325376
I (1224) descent-display: RGB scanout started
I (1234) descent: Heap after display init: total=4453931 internal=12839 DMA=5131 PSRAM=4441092 largest_internal=7680 largest_DMA=4864 largest_PSRAM=4325376
I (1244) descent: Heap before POST: total=4453931 internal=12839 DMA=5131 PSRAM=4441092 largest_internal=7680 largest_DMA=4864 largest_PSRAM=4325376
BLE HID: scan found 0 BLE HID device(s)
BLE HID: scan found 1 device(s), opening match 'Xbox Wireless Controller'
I (15054) ESP_HID_GAP: BLE GAP AUTH SUCCESS
BLE HID: connected to 'Xbox Wireless Controller'
I (20604) descent-cdrom: Descent CD image found: /sdcard/descent.iso
I (20604) descent-post: DESCENT.HOG: PASS
I (20604) descent-post: DESCENT.PIG: PASS
I (20604) descent-post: DATA: REGISTERED 1.0 REF; PIG: EARLY D1 LAYOUT
I (20644) descent: Heap after POST: total=4442143 internal=4723 DMA=3759 PSRAM=4437420 largest_internal=3072 largest_DMA=3072 largest_PSRAM=4325376
I (20654) descent: Battery init: gauge probe begin
I (20654) descent: Battery task create: internal=10975 largest=6144
I (20664) descent: Battery task create result=1 (PASS)
I (20664) descent: Heap before INFERNO: total=4444127 internal=6771 DMA=3759 PSRAM=4437356 largest_internal=3072 largest_DMA=3072 largest_PSRAM=4325376
I (20674) descent: LC709203F found, starting 10-second poll
I (20674) descent: Entering INFERNO with data directory /sdcard/DESCENT/
I (20684) descent: Batt: 99.4% (4.17V)
Text: 555 original entries + 66 later-version compatibility entries

DESCENT   Registered v1.5 Jan 5, 1996
Copyright (C) 1994, 1995 Parallax Software Corporation
DESCENT is a trademark of Interplay Productions, Inc.

Type 'DESCENT -help' for a list of command-line options.

[MONO 0: Debug Spew]

[MONO 1: Errors & Serious Warnings]
WVIDEO_running = 0
Getting settings from DESCENT.CFG...
Initializing timer system...
Initializing keyboard handler...
Initializing mouse handler...I (20924) mouse-touch: GT911 at 0x14: 800x480; touch = left mouse

Initializing joystick handler...
Initializing divide by zero handler...
Initializing network... No IPX compatible network found.
Network support disabled...

Initializing graphics system...

STACK before gr_init: 5312
STACK after gr_init: 5312
Going into graphics mode...STACK after gr_set_mode: 5312

Initializing palette system...
PAL: enter, stack=5312
PAL: cfopen
PAL: cfopen returned 0x600ffde0, stack=5312
PAL: cfilelength
PAL: length=9472, stack=5312
PAL: read palette
PAL: palette read, stack=5312
PAL: read fade table
PAL: fade read, stack=5312
PAL: close
PAL: patch fade table
PAL: done, stack=5312
STACK after palette: 5312

Initializing font system...STACK before gamefont_init: 5312
STACK after gamefont_init: 3280
I (30694) descent: Batt: 99.4% (4.17V)
PIG: early D1 directory, 1607 bitmaps, 98 sounds

Error: Early D1 PIG directory loaded; BITMAPS.BIN/TBL game-definition reader (BMREAD) still needs porting

abort() was called at PC 0x42009252 on core 0
--- 0x42009252: _exit at C:/esp/v6.0.1/esp-idf/components/esp_libc/src/syscalls.c:121


Backtrace: 0x40389391:0x3fcd4950 0x4038935d:0x3fcd4970 0x40387eda:0x3fcd4990 0x42009252:0x3fcd4a00 0x420c6aae:0x3fcd4a20 0x420b59c5:0x3fcd4a40 0x4202b386:0x3fcd4a90 0x42015173:0x3fcd4ab0 0x42010fdd:0x3fcd4df0 0x42008c03:0x3fcd4e30
--- 0x40389391: panic_abort at C:/esp/v6.0.1/esp-idf/components/esp_system/panic.c:464
--- 0x4038935d: esp_system_abort at C:/esp/v6.0.1/esp-idf/components/esp_system/port/esp_system_chip.c:87
--- 0x40387eda: abort at C:/esp/v6.0.1/esp-idf/components/esp_libc/src/abort.c:38
--- 0x42009252: _exit at C:/esp/v6.0.1/esp-idf/components/esp_libc/src/syscalls.c:121
--- 0x420c6aae: exit at /builds/idf/crosstool-NG/.build/xtensa-esp-elf/build/build-picolibc-build-x86_64-build_pc-linux-gnu/../../../src/picolibc-git-1738c754/newlib/libc/stdlib/pico-exit.c:49
--- 0x420b59c5: Error(char const*, ...) at C:/Projects/dESPcent/source/MISC/ERROR.cpp:116
--- 0x4202b386: bm_init() at C:/Projects/dESPcent/source/MAIN/BM.cpp:90
--- 0x42015173: descent_main(int, char**) at C:/Projects/dESPcent/source/MAIN/INFERNO.cpp:1031
--- 0x42010fdd: app_main at C:/Projects/dESPcent/source/MAIN/main.cpp:220
--- 0x42008c03: main_task at C:/esp/v6.0.1/esp-idf/components/freertos/app_startup.c:199




ELF file SHA256: 273036250

Rebooting...
```

### Current startup blocker

```
// Initializes all bitmaps from BITMAPS.TBL file.
int bm_init()
{
	init_polygon_models();
	piggy_init(); // This calls bm_read_all
	if (piggy_requires_bitmap_table())
		Error("Early D1 PIG directory loaded; BITMAPS.BIN/TBL game-definition reader (BMREAD) still needs porting");
	piggy_read_sounds();
	return 0;
}
```

The current halt occurs immediately after piggy_init() returns. Early Descent data has been identified successfully, but the original BMREAD path that populates the bitmap/game-definition tables from BITMAPS.BIN / BITMAPS.TBL has not yet been ported.

The next step is to port the early-D1 BMREAD/game-definition path so bitmap metadata can be populated and initialization can continue beyond bm_init().

Menus, level rendering, playable controls, sound, and music are not demonstrated by this startup log. There is no measured gameplay frame rate yet.

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

The battery and gauge are additions to this prototype. Their presence does not establish battery runtime, which has not been measured here.

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
