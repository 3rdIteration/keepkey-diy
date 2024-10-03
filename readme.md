# Keepkey DIY

# Hardware Required

## MCU Options
Basically any board that works well with Trezor One is good here. STM32F2 or STM32F4 board with 1MB of flash will probably work...

STM32F405RGT6 works well and you can a very good low cost board that is cheap, compatible and has a USB-C connector.

You can get it here (be sure to select the STM32F405RGT6 variant)
https://www.aliexpress.com/item/1005006051651136.html 

## Display Options
To run with the stock Keepkey firmware, you need a display that is:
* 256x64 Resolution
* SSD1322 Controller
* 8080 Parallel Connection Support

You can get either a 5.5 or 3.12 inch display that meets these requirements.

The following two displays work fine:

https://www.aliexpress.com/item/1005006150123067.html

https://www.aliexpress.com/item/1005005985371717.html

## Input Button
The GPIO setup for the Keepkey doesn't make use of an internal pull-up resistor, so one will need to be required.

Some three pin digital push-buttons include, so can generally be used for this. (Or any two pin button if you add an external 10k pull-up resistor)

## 3d printed case:
3.12 inch version https://www.printables.com/model/1027257-case-for-keepkey-diy-312-inch-display
5.5 inch version https://www.printables.com/model/1027254-case-for-keepkey-diy-55-inch-display

# Assembly
Some of the display modules for don't include pin labels, only numbers.

## Display

| STM32 IO | Display Description | Display Pin Number (For 16 pin) |
|----------|-------------------|---------------------------------|
| GND      | GND               | 1                               |
| 3.3v     | VCC               | 2                               |
| Not connected | Not Connected | 3 |
| A0       | D0                | 4                               |
| A1       | D1                | 5                               |
| A2       | D2                | 6                               |
| A3       | D3                | 7                               |
| A4       | D4                | 8                               |
| A5       | D5                | 9                               |
| A6       | D6                | 10                              |
| A7       | D7                | 11                              | 
| A8       | E/RD              | 12                              |
| A9       | R/W               | 13                              |
| B1       | D/C               | 14                              |
| B5       |RES                | 15                              |
|A10       |CS                 | 16                              |

## Button
Connect to STM32 pin B7

You need to provide your own Pullup/Pulldown setup for it if you want to run stock firmware…( If you don't, it won't register presses properly)

For a 3 pin button like the one here: https://www.aliexpress.com/item/1005005637908638.html

For these buttons, connect STM32-3.3v to button-GND (This ties the
input high via a 10k resistor), STM32-B7 to Button-Out and STM32-GND to
button-VCC)

# Software

## Debug Build and Flash
Build as per official documentation with Docker… (https://github.com/keepkey/keepkey-firmware/tree/master)

Before attempting to install stock firmware (which locks the bootloader and disables the programming interface) you should build and flash the debug firmware first.

The key difference from the official documentation is:

    ./scripts/build/docker/device/debug.sh

Install STM32 Cube Programmer
Put the board in DFU mode then connect… (Hold down B0)
Flash using STM32 Cube Programmer with the following steps. (Uncheck run afterward)
1. Full flash erase
2. Uncheck "Run after programming"
3. Flash the bootstrap.bin @ 0x08000000
4. Flash relevant variant.VARIANT_NAME.bin @ 0x08010000 (The logo that is used, you could customise this if you want)
5. Flash bootloader.bin @ 0x08020000
6. Flash firmware.keepkey.bin @ 0x08060000

Restart device…

## Flashing Locked Firmware
If you flash a release build of the bootloader, it will secure the device and make it so that you are then able to load official Keepkey firmware onto your DIY device. (Providing equivalent security and functionality of a retail Keepkey)

### Building a specific bootloader version
The version tags for normal firmware are not the same as the versions for the bootloader releases. As such you need to first checkout a bootloader release (these are tagged with a tag name staring with bl)

### Checking Bootloader Hash
It is *critical* that the bootloader hash matches one of the values listed here: https://github.com/keepkey/keepkey-desktop/blob/develop/firmware/releases.json#L32

The entries in this file are not simply a single SHA256 checksum, but a double SHA256 of the bootloader binary. (So basically the HEX output of the SHA256 of the file, is hashed a second time to produce the checksum listed above)

### Flashing the firmware 
Use the same process as was used with the debug firmware, substituting the firmware file with the latest official release. 

If you accidentally flash unsigned firmware then you will need to use the Keepkey uodater tool to flash the latest official firmware.
