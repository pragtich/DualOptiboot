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
