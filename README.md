Local changes to DualOptiboot
============================

- Uses Kristerw/kisse66's modifications to use an I2C EEPROM (make atmega328_e8)
- Flash using [an ISP programmer](https://www.pragti.ch/computer%20stuff/2018/09/10/Uploading_bootloader_Pro_Mini/).

Flashing
========

make sure `PATH` contains the compiler and tools, so when I want to use Platformio's tools:
` export PATH=$PATH:~/.platformio/packages/toolchain-atmelavr/bin:~/.platformio/packages/tool-avrdude`

Check out [Adafruit's tutorial on avrdude](http://www.ladyada.net/learn/avr/avrdude.html). Also useful is [Adafruit's explanation of using the ArduinoISP](https://learn.adafruit.com/arduino-tips-tricks-and-techniques/arduinoisp).


```sh
make atmega328_e8

# Connect to ISP

# 1.8V brownout 2048 word bootloader (starts 0x3800)
avrdude  -C ~/.platformio/packages/tool-avrdude/avrdude.conf  -p atmega328p -c arduino -P /dev/ttyUSB0 -b 19200 -U flash:w:optiboot_atmega328p_e8.hex  -U hfuse:w:0xD8:m -U lfuse:w:0xFF:m -U efuse:w:0xFE:m 

# same but 1024 word bootloader (starts 0x3c00)
avrdude  -C ~/.platformio/packages/tool-avrdude/avrdude.conf  -p atmega328p -c arduino -P /dev/ttyUSB0 -b 19200 -U flash:w:optiboot_atmega328p_e8.hex  -U hfuse:w:0xDA:m -U lfuse:w:0xFF:m -U efuse:w:0xFE:m 

# forum version: Assumes start=.text=0x7a00
avrdude  -C ~/.platformio/packages/tool-avrdude/avrdude.conf  -p atmega328p -c arduino -P /dev/ttyUSB0 -b 19200 -U flash:w:optiboot_atmega328_e8.hex  -U hfuse:w:0xDA:m -U lfuse:w:0xFF:m -U efuse:w:0x05:m

```

Debugging flash problems:

- Upload of dualoptiboot is working (acc to debug)
- Flashinng with avrdude fails (no response)
- Sigrok: avrdude is sending 30 20 at 115200 baud. No response!

Test upload from arduino examples folder:
`/home/pragtich/.platformio/platforms/atmelavr/examples/arduino-blink`
and then perform:
` avrdude -v -v -v -v -C ~/.platformio/packages/tool-avrdude/avrdude.conf  -p atmega328p -c stk500v1 -P /dev/ttyUSB0 -b 115200 -U flash:w:.pioenvs/pro8MHzatmega328/firmware.hex`

Result with debug (annoyingly). I configured the baud rate to 115200

``` 
SSF2XASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASASSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSS
```

Finally a working solution:
[In github](https://github.com/pragtich/DualOptiboot/commit/76f8e6908119444459f8283d492ce6472d8d52fc)
I am not sure what I did wrong, except that it seems that the bootloader might not start properly without a proper application to run. When I reverted to the most basic setup and manually merged a blink script into the hex file (merge and delete the EOF line), it seems to work now.

TODO:

- [x] Test a real mysensors script
- [ ] Test FOTA on this script
- [ ] Test the EEPROM FOTA stuff
- [ ] Understand why I stuffed up for so long
- [ ] Update to the newest Optiboot base
- [ ] remake repository structure
- [ ] Document experience


# Test a real mysensors script

avrdude  -C ~/.platformio/packages/tool-avrdude/avrdude.conf  -p atmega328p -c arduino -P /dev/ttyUSB0 -b 19200  -U hfuse:w:0xD8:m -U lfuse:w:0xFF:m -U efuse:w:0xFE:m -U flash:w:optiboot57k6.hex

Edit platformio.ini to contain:

upload_port = /dev/ttyUSB0
upload_speed = 57600
monitor_speed = 9600
monitor_port = /dev/ttyUSB0

pio run -t upload
pio device monitor

--- Miniterm on /dev/ttyUSB0  9600,8,N,1 ---
--- Quit: Ctrl+C | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---
NodeManager v1.8-dev
LIB V=2.3.1-alpha R=R E=- T=N A=A S=- B=-
BATTERY I=201 P=30 T=38
SIGNAL I=202 P=33 T=37
SIGNAL I=300 P=33 T=37
DOOR I=1 P=0 T=16
RADIO...


Works...

# Make it really do a FOTA

Add to the sketch:


#define USE_I2C_EEPROM
#define MY_OTA_FIRMWARE_FEATURE
#define MY_OTA_USE_I2C_EEPROM 

Needed to reduce size due to crash loop?

Compile error:
Compiling .pioenvs/pro8MHzatmega328/src/Deur02.ino.cpp.o
In file included from lib/MySensors/drivers/I2CEeprom/I2CEeprom.cpp:31:0,
from lib/MySensors/MySensors.h:135,
from src/NodeManagerLibrary.h:146,
from /home/pragtich/mysensors/Sensors/deur02/src/Deur02.ino:181:
lib/MySensors/drivers/I2CEeprom/I2CEeprom.h:49:23: fatal error: extEEPROM.h: No such file or directory

- [ ] TODO: make platformio find the right header file; It's there.
- [ ] TODO: make the bootloader and my sketch work at the same baudrate

pio lib install 804


Boot loop again?!

Reflash bootloader with ISP (disconnect radio) and try again; Reproducible

Remove even more code from Node: Success!

- [ ] TODO: Understand NodeManager size and reduce
- [ ] TODO: Fix symlinks to cloned folders. I've got out-of-date copies everywhere...

DualOptiboot
============

Custom Optiboot to add wireless programming capability to Moteino
Copyright Felix Rusu (2013-2014), felix@lowpowerlab.com
More at: http://lowpowerlab.com/Moteino

This Optiboot version is modified to add the capability of reflashing 
from an external SPI flash memory chip. As configured this will work 
with Moteino (www.lowpowerlab.com/Moteino) provided a SPI flash chip
is present on the dedicated onboard footprint.
Summary of how this Optiboot version works:
- it looks for an external flash chip
- if one is found (SPI returns valid data) it will further look
  for a new sketch flash image signature and size
  starting at address 0:   FLXIMG:9999:XXXXXXXXXXX
  where: - 'FLXIMG' is fixed signature indicating FLASH chip
           contains a valid new flash image to be burned
         - '9999' are 4 size bytes indicating how long the
           new flash image is (how many bytes to read)
         - 'XXXXXX' are the de-hexified bytes of the flash 
           pages to be burned
         - ':' colons have fixed positions (delimiters)
- if no valid signature/size are found, it will skip and
  function as it normally would (listen to STK500 protocol on serial port)

The added code will result in a compiled size of just under 1kb
(Originally Optiboot takes just under 0.5kb)

-------------------------------------------------------------------------------------------------------------

To compile copy the Optiboot.c and Makefile files where Optiboot is originally located, mine is at:
arduino-install-dir\hardware\arduino\bootloaders\optiboot\
Backup the original files andbefore overwrite both files.
Then compile by running:
make atmega328
make atmega1284p

##License
GPL 3.0. See License.txt file.
