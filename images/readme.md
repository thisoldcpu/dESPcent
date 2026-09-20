# dESPcent: From Boot to Descent

A development gallery documenting the port of the original Descent to a portable ESP32-S3 system. Original game code, original game data, and a new home on a 7-inch screen.

## Startup POST on Hardware

[![dESPcent running on a CrowPanel display, with board diagnostics on the left and Descent SD card file checks on the right.](/assets/images/despcent_post_screen.jpg)](/assets/images/despcent_post_screen.jpg)

*The board boots, the display works, and the required Descent archives are readable from the SD card. Click the photo to view it at full resolution.*

The left panel reports the ESP32-S3, memory availability, display configuration, and SD card status. The right panel checks `DESCENT.HOG` and `DESCENT.PIG` in `/DESCENT`, with a short description of what each archive supplies.

Both required files pass the readability check in this photograph. An ISO is also detected and listed as optional; it is not used for runtime. This image captures the milestone before data version detection was added.

## The Platform

- **Board:** Elecrow CrowPanel 7-inch, ESP32-S3
- **Display:** 800 × 480, RGB565
- **Memory:** 4 MB flash and 8 MB PSRAM
- **Game data:** Original Descent archives on SD card
- **Milestone pictured:** Working startup POST and required-file checks

## Following the Original

The next stage connects our board startup to Descent's original entry sequence. The aim is to port that sequence faithfully, working through its dependencies toward the opening logos, menus, and eventually the game itself.
