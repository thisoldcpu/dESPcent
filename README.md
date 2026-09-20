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

## Target Hardware

Initial development target:

The dESPcent project targets the Elecrow CrowPanel 7.0" HMI ESP32 Display, powered by the ESP32-S3-WROOM-1-N4R8 module. While modern microcontrollers offer clock speeds that easily exceed mid-90s desktop PCs, translating a fully 3D, six-degrees-of-freedom engine like *Descent* to an embedded architecture requires navigating fundamental differences in memory behavior, storage interfaces, and display scanning techniques. 

## Hardware Specification Comparison

| Component | 1995 Minimum | 1995 Recommended | dESPcent (CrowPanel ESP32-S3) |
| :--- | :--- | :--- | :--- |
| **Processor** | Intel 386 @ 33 MHz | Intel 486 @ 33–66 MHz | ESP32-S3 (Dual-core Xtensa @ 240 MHz) |
| **Volatile Memory** | 4 MB RAM | 8 MB RAM | 8 MB PSRAM |
| **Non-Volatile** | 20 MB Hard Drive | 20 MB Hard Drive | 4 MB SPI Flash + 32GB SD Card |
| **Display/Video** | VGA (320x200) | SVGA (640x480) | 7.0" 800x480 RGB Panel |
| **Audio** | Sound Blaster/AdLib | Sound Blaster 16 | I2S via NS4168 Power Amplifier |
| **Power/Portability** | AC Desktop Power | AC Desktop Power | 3700mAh LiPo & Adafruit LC709203F Battery Gauge |

## Memory Architecture: The 8MB Illusion

Matching the original 8 MB recommended memory footprint with the ESP32-S3's 8 MB PSRAM creates an illusion of parity. However, the architectural realities necessitate strict memory management strategies:

*   **8MB EDO DRAM is not PSRAM:** In 1995, EDO (Extended Data Out) DRAM provided low-latency, parallel access directly over the motherboard's memory bus. The ESP32-S3 relies on serial SPI-based PSRAM. While PSRAM provides the necessary capacity, serializing memory requests over a bus introduces cache-miss latency penalties that did not exist on a 486 processor.
*   **EDO DRAM is not Flash:** Original DOS gaming loaded assets from the hard drive directly into fast, executable RAM. On the ESP32-S3, the 4 MB onboard Flash executes code via XIP (eXecute In Place). Flash memory offers significantly lower throughput than PSRAM or 1990s DRAM. Performance-critical engine loops and lookup tables must be aggressively pinned to the ESP32-S3's small internal SRAM to avoid bus contention between fetching instructions from Flash and reading level geometry from PSRAM.
*   **Asset Streaming & Storage:** The original game relied on a fast-spinning hard drive (relative to its era). dESPcent utilizes a 32GB SD Card connected via a 4-pin SPI interface (MOSI / MISO / SCLK / CS mapping to GPIOs 11, 13, 12, and 10). Streaming bulk level data, textures, and audio concurrently demands careful DMA scheduling to prevent stuttering.

## Display Dynamics and Framebuffer Footprints

The continuous scanning requirements of the CrowPanel's 800x480 RGB display present the largest departure from vintage VGA rendering. The ESP32-S3 uses a continuously scanned RGB interface, which differs completely from older command/data interfaces.

*   **Bandwidth Overhead:** A conventional 800x480 RGB565 framebuffer consumes 768,000 bytes. Implementing double buffering costs 1,536,000 bytes, instantly claiming nearly 20% of the available PSRAM. 
*   **Active Scanout Limits:** At the calculated panel timing of ~30.79 refreshes per second, active scanout requires approximately 23.65 MB/s of pixel payload bandwidth. This massive data movement occurs before the CPU performs any rendering writes, potentially starving other memory users.
*   **Rendering Compromises:** To respect bandwidth limits and optimize PSRAM bus load, dESPcent targets a smaller 384x240 indexed engine framebuffer, which only requires 92,160 bytes. This low-resolution buffer will be upscaled to a 768x480 active area with 16-pixel side borders. 

## Hardware Boons and Modern Integration

Despite the architectural bottlenecks, the ESP32-S3 platform introduces significant quality-of-life integrations unseen in the 1995 PC ecosystem:

*   **Compact Self-Sufficiency:** A fully self-contained portable unit is achieved via the JST 2 jumper cable, MakerFocus 2700mAh LiPo battery, and I2C fuel gauge.
*   **Modern Audio I/O:** The built-in NS4168 amplifier manages I2S digital audio natively. By routing the SDIN to GPIO17, LRCLK to GPIO18, and BCLK to GPIO42, the system completely replaces legacy sound cards and eliminates vintage IRQ conflicts.
*   **Capacitive Touch:** The integrated GT911 capacitive touch interface—managed via the PCA9557 I2C expander for reset and address-selection timing opens up modern UI interaction paradigms for menus without requiring a bulky keyboard and mouse.

Other ESP32-S3 hardware may be supported later where practical.

## Display Strategy

The 800×480 panel is the **output resolution**, not necessarily the rendering resolution.

The original Descent renderer was designed for considerably more constrained hardware than a modern PC, making its software renderer and fixed-point mathematics particularly interesting on a microcontroller.

Initial targets will likely include resolutions such as:

- 320×200
- 320×240
- 384×240

A 384×240 framebuffer can be scaled 2× to **768×480**, leaving only narrow side borders on the 800×480 display while requiring the engine to render less than one quarter of the panel's native pixel count.

## Development Philosophy

Accuracy and functionality come first.

The initial objective is not to rewrite or heavily simplify Descent until it fits. Instead, we'll establish a working baseline, identify the actual CPU, memory, rendering, and I/O bottlenecks, and optimize based on measurements.

If something is slow, we want to know **why** it's slow.

If something has to be changed for the ESP32-S3, that change should have a measurable reason for existing.

## Current Status

**Very early development.**

Right now, dESPcent is an experiment:

> Can an ESP32-S3 run a useful native port of Descent?

~~We don't know yet. That's the fun part.~~

**It's alive!**

The core Descent engine natively boots on the ESP32-S3. It successfully mounts the SD card, initializes Bluetooth for wireless controllers, and passes the initial `DESCENT.HOG` and `DESCENT.PIG` asset checks. 

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
I (1133) BLE_INIT: Bluetooth MAC: 74:4d:bd:9d:91:b6
I (1133) phy_init: phy_version 711,97bcf0a2,Aug 25 2025,19:04:10
I (1183) descent: Heap after BLE begin: total=4459623 internal=18607 DMA=10819 PSRAM=4441016 largest_internal=10752 largest_DMA=10752 largest_PSRAM=4325376
I (1193) descent: BLE controller ready; scanning for Xbox controller
BLE: 20:64:de:b2:84:d8, RSSI: -60, UUID: 0x0000, APPEARANCE: 0x0000, ADDR_TYPE: 'PUBLIC', NAME: 'J's Charge 4'
I (1203) descent: Heap after BLE init: total=4453283 internal=12319 DMA=4531 PSRAM=4440964 largest_internal=7680 largest_DMA=4352 largest_PSRAM=4325376
I (1213) descent: Heap before display init: total=4453283 internal=12319 DMA=4531 PSRAM=4440964 largest_internal=7680 largest_DMA=4352 largest_PSRAM=4325376
BLE: 74:6d:fa:71:1f:d9, RSSI: -67, UUID: 0x0000, APPEARANCE: 0x0000, ADDR_TYPE: 'PUBLIC'
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

The engine is currently executing real 1995 code and immediately crashing because it thinks it's running into the memory wall. The next major hurdle is bypassing these legacy DOS environment checks and bringing up the software renderer.

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
