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

# Debugging sketch size

A simple NodeManager script (NodeManager $master$ aka $v1.7$. Mysensors $development$ version $5d159a6c57209e9be91834b498d0072d3a1a25d6$.

Simple platformio compile of a basic NodeManager script (deur02 version $7ed71ad6d820cc598ad607d3e543d65d153c852d $) gives the following sizes. This is way too large!

```
Linking .pioenvs/pro8MHzatmega328/firmware.elf
Checking size .pioenvs/pro8MHzatmega328/firmware.elf
Building .pioenvs/pro8MHzatmega328/firmware.hex
Memory Usage -> http://bit.ly/pio-memory-usage
DATA:    [=====     ]  48.9% (used 1001 bytes from 2048 bytes)
PROGRAM: [========= ]  94.1% (used 28920 bytes from 30720 bytes)
```

This is forgetting about my debug bootloader, which uses 1024 words (2048 bytes) and thus will lead to conflicts with a program larger than 30k.

What is in this sketch that's so difficult? Perhaps:

- No tuning of NodeManager for size
- I2C EEPROM FOTA is enabled
- DEBUG is on
- Local handlers enabled in order to give periodic updates (hooking)

Let's try enabling/disabling some stuff:

## debug disabled

Dependency Graph
|-- <SD> 1.1.1
|   |-- <SPI> 1.0
|-- <SPI> 1.0
|-- <WiFi> 1.2.7
|   |-- <SPI> 1.0
|-- <Wire> 1.0
|-- <MySensors> 2.3.1-beta
|   |-- <EEPROM> 2.0
|   |-- <Wire> 1.0
|   |-- <extEEPROM> 3.4
|   |   |-- <Wire> 1.0
|   |-- <WiFi> 1.2.7
|   |   |-- <SPI> 1.0
|   |-- <SPI> 1.0
|-- <SoftwareSerial> 1.0
|-- <Servo> 1.1.2
Compiling .pioenvs/pro8MHzatmega328/src/Deur02.ino.cpp.o
/home/pragtich/mysensors/Sensors/deur02/src/NodeManagerLibrary.ino: In member function 'void Sensor::before()':
/home/pragtich/mysensors/Sensors/deur02/src/NodeManagerLibrary.ino:658:12: warning: unused variable 'child' [-Wunused-variable]
Child* child = *itr;
^
Linking .pioenvs/pro8MHzatmega328/firmware.elf
Checking size .pioenvs/pro8MHzatmega328/firmware.elf
Building .pioenvs/pro8MHzatmega328/firmware.hex
Memory Usage -> http://bit.ly/pio-memory-usage
DATA:    [====      ]  43.8% (used 896 bytes from 2048 bytes)
PROGRAM: [======    ]  59.3% (used 18208 bytes from 30720 bytes)


## OTA and I2C disabled

Dependency Graph
|-- <SPI> 1.0
|-- <SD> 1.1.1
|   |-- <SPI> 1.0
|-- <Wire> 1.0
|-- <WiFi> 1.2.7
|   |-- <SPI> 1.0
|-- <MySensors> 2.3.1-beta
|   |-- <EEPROM> 2.0
|   |-- <Wire> 1.0
|   |-- <extEEPROM> 3.4
|   |   |-- <Wire> 1.0
|   |-- <WiFi> 1.2.7
|   |   |-- <SPI> 1.0
|   |-- <SPI> 1.0
|-- <SoftwareSerial> 1.0
|-- <Servo> 1.1.2
Compiling .pioenvs/pro8MHzatmega328/src/Deur02.ino.cpp.o
Linking .pioenvs/pro8MHzatmega328/firmware.elf
Checking size .pioenvs/pro8MHzatmega328/firmware.elf
Building .pioenvs/pro8MHzatmega328/firmware.hex
Memory Usage -> http://bit.ly/pio-memory-usage
DATA:    [====      ]  37.1% (used 759 bytes from 2048 bytes)
PROGRAM: [========  ]  81.5% (used 25032 bytes from 30720 bytes)

Why is Wire still a dependency?? And WiFi? And SoftwareSerial? And Servo?

To be checked: are they actually linked in?

## Finer comparison

avr-nm -Crtd --size-sort bin/fw+dbg+ota.elf

00002980 T transportProcess()
00001468 t global constructors keyed to 65535_0_Deur02.ino.cpp.o.3111
00000932 T vfprintf
00000830 T ChildInt::sendValue(bool)
00000822 T stInitUpdate()
00000796 T dtoa_prf
00000698 T strtod
00000684 T _sleep(unsigned long, bool, unsigned char, unsigned char, unsigned char, unsigned char)
00000604 T __vector_24
00000590 t Sensor::loop(MyMessage*)
00000498 t before()
00000494 T loop
00000488 T main
00000452 T extEEPROM::write(unsigned long, unsigned char*, unsigned int) [clone .part.1] [clone .constprop.20]
00000444 T setup
00000432 T __ftoa_engine
00000368 T RFM69::interruptHandler()
00000328 t transportSendWrite(unsigned char, MyMessage&)
00000324 T presentation()
00000314 T MyMessage::getString(char*) const [clone .constprop.48]
00000304 T free
00000298 T malloc
00000268 T SensorInterrupt::onInterrupt()
00000266 T RFM69::sendFrame(unsigned char, void const*, unsigned char, bool, bool)
00000216 T hwSleep(unsigned char, unsigned char, unsigned char, unsigned char, unsigned long)
00000210 T __mulsf3x
00000204 T __addsf3x
00000198 T RFM69::sendACK(void const*, unsigned char)
00000194 T RFM69::sendWithRetry(unsigned char, void const*, unsigned char, unsigned char, unsigned char)
00000188 T __ultoa_invert
00000186 T transportRouteMessage(MyMessage&)
00000184 T TwoWire::endTransmission(unsigned char)
00000184 T presentNode()
00000166 t NodeManager::_onInterrupt_2()
00000166 t NodeManager::_onInterrupt_1()
00000166 t stUplinkUpdate()
00000166 T hwPowerDown(unsigned char)
00000162 T RFM69::setMode(unsigned char)
00000162 T receive(MyMessage const&)
00000158 T RFM69::send(unsigned char, void const*, unsigned char, bool)
00000157 B Serial
00000148 T __vector_16
00000146 T Print::printNumber(unsigned long, unsigned char)
00000142 T HardwareSerial::write(unsigned char)
00000130 T hwInternalSleep(unsigned long)
00000130 T hwDebugPrint(char const*, ...)
00000122 T __floatunsisf
00000120 t stIDTransition()
00000120 T pinMode
00000116 T stParentUpdate()
00000112 T fputc
00000110 T transportAssignNodeID(unsigned char)
00000108 T wait(unsigned long, unsigned char, unsigned char) [clone .constprop.13]
00000106 T MyMessage::set(char const*)
00000102 T SensorInterrupt::onReceive(MyMessage*)
00000100 T Print::print(long, int) [clone .constprop.83]
00000100 T __vector_18
00000098 T RFM69::receiveBegin()
00000098 T _callbackTransportReady()
00000098 T stFailureTransition()
00000096 T TwoWire::write(unsigned char const*, unsigned int)
00000096 T digitalWrite
00000094 T TwoWire::write(unsigned char)
00000094 T __fixunssfsi
00000088 T transportSwitchSM(transportState_t&)
00000088 t stReadyTransition()
00000088 T vsnprintf_P
00000086 T Print::write(unsigned char const*, unsigned int)
00000084 T RFM69::readReg(unsigned char)
00000084 T NodeManager::getSensorWithChild(int)
00000084 t NodeManager::sleepOrWait(long)
00000084 t stIDUpdate()
00000082 t turnOffPWM
00000082 T digitalRead
00000080 T RFM69::writeReg(unsigned char, unsigned char)
00000080 T SensorInterrupt::onSetup()
00000080 T HardwareSerial::begin(unsigned long, unsigned char) [clone .constprop.76]
00000080 T wait(unsigned long)
00000080 T stFailureUpdate()
00000080 t twi_transmit
00000078 T __vector_2
00000078 T __vector_1
00000078 T atol
00000076 T __vector_19
00000075 t transportSendWrite(unsigned char, MyMessage&)::__c
00000074 T micros
00000072 T __fp_cmp
00000070 t RFM69::canSend()
00000069 B node
00000068 T RFM69::receiveDone()
00000068 T HardwareSerial::flush()
00000068 T __ultoa_ncheck
00000068 T __udivmodsi4
00000068 T __fp_split3
00000066 T Print::print(__FlashStringHelper const*) [clone .constprop.79]
00000066 t HardwareSerial::_tx_udr_empty_irq()
00000066 T stParentTransition()
00000064 T I2CEeprom::busy() [clone .constprop.58]
00000064 T RFM69::setHighPower(bool)
00000064 T dtostrf
00000061 B RFM69::DATA
00000060 T Sensor::getChild(int)
00000060 t stUplinkTransition()
00000060 T stInitTransition()
00000060 T atoi
00000058 t transportProcessMessage()::__c
00000056 T __prologue_saves__
00000054 T __epilogue_restores__
00000052 T RFM69::setHighPowerRegs(bool)
00000052 T RFM69::setPowerLevel(unsigned char)
00000052 T NodeManager::_sleepBetweenSend()
00000052 T __utoa_ncheck
00000051 B _convBuf
00000050 T build(MyMessage&, unsigned char, unsigned char, unsigned char, unsigned char, bool) [clone .constprop.84]
00000050 T attachInterrupt
00000048 T present(unsigned char, unsigned char, char const*, bool)
00000048 T strncasecmp_P
00000047 t _sleep(unsigned long, bool, unsigned char, unsigned char, unsigned char, unsigned char)::__c
00000046 t _begin()::__c
00000046 t _GLOBAL__sub_D__Z6hwInitv
00000044 T RFM69::select()
00000044 T Print::println(int, int) [clone .constprop.80]
00000042 T MyMessage::clear()
00000042 T Print::write(char const*)
00000040 T TwoWire::read()
00000040 T HardwareSerial::read()
00000040 T __udivmodhi4
00000040 B door
00000040 T __divmodhi4
00000038 T MyMessage::getByte() const [clone .constprop.47]
00000038 T Print::println(char const*) [clone .constprop.35]
00000038 T transportTimeInState()
00000037 t firmwareOTAUpdateRequest()::__c
00000036 t stReadyUpdate()
00000036 T eeprom_update_byte
00000036 T detachInterrupt
00000034 T __fp_round
00000033 B _msgTmp
00000033 B _msg
00000032 D vtable for RFM69
00000032 b TwoWire::txBuffer
00000032 B TwoWire::rxBuffer
00000032 T Print::println(__FlashStringHelper const*) [clone .constprop.15]
00000032 t wakeUp2()
00000032 t wakeUp1()
00000032 b twi_txBuffer
00000032 b twi_rxBuffer
00000032 B twi_masterBuffer.lto_priv.102
00000032 T strrev
00000032 T eeprom_update_block
00000032 T eeprom_read_block
00000031 t stReadyTransition()::__c
00000030 t transportIsValidFirmware()::__c
00000030 T MyMessage::set(unsigned long)
00000030 T TwoWire::peek()
00000030 T RFM69::unselect()
00000030 T HardwareSerial::availableForWrite()
00000030 T transportSendRoute(MyMessage&)
00000030 T __umulhisi3
00000030 T strncpy
00000030 T __ltoa_ncheck
00000029 t transportProcessMessage()::__c
00000028 t transportProcessMessage()::__c
00000028 t transportProcessMessage()::__c
00000028 T HardwareSerial::peek()
00000026 t firmwareOTAUpdateProcess()::__c
00000026 T transportDisable()
00000026 T memcmp
00000025 t transportProcessMessage()::__c
00000025 t stReadyUpdate()::__c
00000024 t firmwareOTAUpdateProcess()::__c
00000024 t _begin()::__c
00000024 t transportProcessMessage()::__c
00000024 t transportProcessMessage()::__c
00000024 t transportProcessMessage()::__c
00000024 t stUplinkUpdate()::__c
00000024 T MyMessage::set(unsigned char)
00000024 T HardwareSerial::available()
00000024 t pwr_p10
00000024 t pwr_m10
00000024 T millis
00000024 T attachInterrupt.part.0.lto_priv.90
00000023 t send(MyMessage&, bool)::__c
00000023 t transportProcessMessage()::__c
00000022 t firmwareOTAUpdateRequest()::__c
00000022 t transportProcessMessage()::__c
00000022 t transportRouteMessage(MyMessage&)::__c
00000022 t RFM69::isr0()
00000022 T strnlen_P
00000022 T strnlen
00000022 T __muluhisi3
00000022 T __itoa_ncheck
00000022 T __do_global_dtors
00000022 T __do_global_ctors
00000022 T __do_copy_data
00000021 t _firmwareResponse(unsigned int, unsigned char*)::__c
00000021 t _processInternalCoreMessage()::__c
00000021 t transportAssignNodeID(unsigned char)::__c
00000020 t _firmwareResponse(unsigned int, unsigned char*)::__c
00000020 t _firmwareResponse(unsigned int, unsigned char*)::__c
00000020 t stParentUpdate()::__c
00000020 t stInitUpdate()::__c
00000020 t Serial0_available()
00000020 T serialEventRun()
00000020 T __vector_6
00000020 t twi_stop
00000020 t digital_pin_to_timer_PGM
00000020 T digital_pin_to_port_PGM
00000020 T digital_pin_to_bit_mask_PGM
00000019 t _firmwareResponse(unsigned int, unsigned char*)::__c
00000019 t transportRouteMessage(MyMessage&)::__c
00000018 t transportAssignNodeID(unsigned char)::__c
00000018 t stFailureUpdate()::__c
00000018 d vtable for TwoWire
00000018 d vtable for HardwareSerial
00000018 T Print::print(unsigned char, int) [clone .constprop.30]
00000018 T memcpy
00000017 t transportProcessMessage()::__c
00000017 t stFailureTransition()::__c
00000017 t stInitUpdate()::__c
00000016 t firmwareOTAUpdateProcess()::__c
00000016 t _firmwareResponse(unsigned int, unsigned char*)::__c
00000016 t _firmwareResponse(unsigned int, unsigned char*)::__c
00000016 t _sleep(unsigned long, bool, unsigned char, unsigned char, unsigned char, unsigned char)::__c
00000016 t _sleep(unsigned long, bool, unsigned char, unsigned char, unsigned char, unsigned char)::__c
00000016 t _sleep(unsigned long, bool, unsigned char, unsigned char, unsigned char, unsigned char)::__c
00000016 t _sleep(unsigned long, bool, unsigned char, unsigned char, unsigned char, unsigned char)::__c
00000016 t transportWaitUntilReady(unsigned long)::__c
00000016 t transportProcessMessage()::__c
00000016 t transportRouteMessage(MyMessage&)::__c
00000016 t stParentUpdate()::__c
00000016 T TwoWire::available()
00000016 B _flash
00000016 T __do_clear_bss
00000015 t firmwareOTAUpdateProcess()::__c
00000015 t stUplinkUpdate()::__c
00000015 B _ZL12_transportSM.lto_priv.94
00000014 t NodeManager::before()::__c
00000014 t _sleep(unsigned long, bool, unsigned char, unsigned char, unsigned char, unsigned char)::__c
00000014 t _sleep(unsigned long, bool, unsigned char, unsigned char, unsigned char, unsigned char)::__c
00000014 t stFailureTransition()::__c
00000014 t transportSendRoute(MyMessage&)::__c
00000014 t stIDUpdate()::__c
00000014 D vtable for Sensor
00000014 D vtable for SensorInterrupt
00000014 D vtable for SensorDoor
00000014 t SPIClass::transfer(unsigned char)
00000014 T ChildInt::print(Print&)
00000014 T Print::print(char)
00000014 T __subsf3
00000014 T __fp_zero
00000014 T __fp_pscB
00000014 T __fp_pscA
00000013 t _begin()::__c
00000013 t _begin()::__c
00000013 t transportProcessMessage()::__c
00000013 t transportReInitialise()::__c
00000013 t transportDisable()::__c
00000013 t stParentUpdate()::__c
00000013 t _registerNode()::__c
00000012 t transportProcessMessage()::__c
00000012 t stUplinkUpdate()::__c
00000012 t stIDTransition()::__c
00000012 T ChildInt::reset()
00000012 T Print::print(int, int) [clone .constprop.82]
00000012 B Wire
00000012 T __tablejump2__
00000012 B _radio
00000012 T __fp_inf
00000011 t stIDUpdate()::__c
00000010 t stParentTransition()::__c
00000010 t stInitTransition()::__c
00000010 d vtable for ChildInt
00000010 d vtable for Child
00000010 T Print::print(char const*) [clone .constprop.37]
00000010 T __unordsf2
00000010 t port_to_output_PGM
00000010 T port_to_mode_PGM
00000010 t port_to_input_PGM
00000010 T __gesf2
00000010 T __cmpsf2
00000010 T abort
00000009 t NodeManager::before()::__c
00000009 t stUplinkTransition()::__c
00000008 t Sensor::presentation()::__c
00000008 t NodeManager::receive(MyMessage const&)::__c
00000008 t NodeManager::_sendMessage(int, int)::__c
00000008 t stIDTransition()::__c
00000008 B _ZL19_nodeFirmwareConfig.lto_priv.92
00000008 T __mulsf3
00000008 T atof
00000007 t NodeManager::_sleep()::__c
00000007 t NodeManager::_sleep()::__c
00000007 t NodeManager::before()::__c
00000007 t NodeManager::setupInterrupts()::__c
00000007 t NodeManager::setupInterrupts()::__c
00000007 t NodeManager::_onInterrupt_2()::__c
00000007 t NodeManager::_onInterrupt_1()::__c
00000006 t NodeManager::_sleep()::__c
00000006 t NodeManager::setup()::__c
00000006 t NodeManager::presentation()::__c
00000006 T Print::availableForWrite()
00000006 T __fp_nan
00000005 t NodeManager::_sleep()::__c
00000005 t NodeManager::_onInterrupt_2()::__c
00000005 t NodeManager::_onInterrupt_1()::__c
00000005 t pstr_inity
00000004 t Sensor::before()::__c
00000004 t Sensor::before()::__c
00000004 t Sensor::before()::__c
00000004 t Sensor::presentation()::__c
00000004 t Sensor::presentation()::__c
00000004 t SensorInterrupt::onInterrupt()::__c
00000004 t SensorInterrupt::onInterrupt()::__c
00000004 t SensorInterrupt::onInterrupt()::__c
00000004 t NodeManager::receive(MyMessage const&)::__c
00000004 t NodeManager::receive(MyMessage const&)::__c
00000004 t NodeManager::receive(MyMessage const&)::__c
00000004 t NodeManager::receive(MyMessage const&)::__c
00000004 t NodeManager::before()::__c
00000004 t NodeManager::before()::__c
00000004 t NodeManager::before()::__c
00000004 t NodeManager::before()::__c
00000004 t NodeManager::before()::__c
00000004 t NodeManager::before()::__c
00000004 t NodeManager::setup()::__c
00000004 t NodeManager::setupInterrupts()::__c
00000004 t NodeManager::setupInterrupts()::__c
00000004 t NodeManager::_sendMessage(int, int)::__c
00000004 t NodeManager::_sendMessage(int, int)::__c
00000004 t NodeManager::_sendMessage(int, int)::__c
00000004 t NodeManager::_sendMessage(int, int)::__c
00000004 t NodeManager::_sendMessage(int, int)::__c
00000004 t NodeManager::_sendMessage(int, int)::__c
00000004 B NodeManager::_last_interrupt_2
00000004 B NodeManager::_last_interrupt_1
00000004 d stFailure
00000004 D _ZL8stUplink.lto_priv.104
00000004 D _ZL8stParent.lto_priv.106
00000004 d stReady
00000004 D _ZL6stInit.lto_priv.103
00000004 d stID
00000004 b _firmwareLastRequest
00000004 B _ZL16_transportConfig.lto_priv.88
00000004 b timer0_overflow_count
00000004 b timer0_millis
00000004 D intFunc.lto_priv.89
00000004 T __cxa_pure_virtual
00000003 t NodeManager::presentation()::__c
00000003 t pstr_nan
00000003 t pstr_inf
00000002 t NodeManager::_sleep()::__c
00000002 t hwDebugPrint(char const*, ...)::__c
00000002 T TwoWire::flush()
00000002 T Sensor::onReceive(MyMessage*)
00000002 T Sensor::onBefore()
00000002 T Sensor::onSetup()
00000002 T Sensor::onLoop(Child*)
00000002 T Sensor::onInterrupt()
00000002 b RFM69::RSSI
00000002 T RFM69::interruptHook(unsigned char)
00000002 b RFM69::selfPointer
00000002 T Print::flush()
00000002 T Child::sendValue(bool)
00000002 T Child::reset()
00000002 T Child::print(Print&)
00000002 T SensorInterrupt::onLoop(Child*)
00000002 b NodeManager::_last_interrupt_value
00000002 d NodeManager::_last_interrupt_pin
00000002 b _firmwareBlock
00000002 B _ZL11_coreConfig.lto_priv.91
00000002 b twi_onSlaveTransmit
00000002 b twi_onSlaveReceive
00000002 B _transportReady_cb
00000002 t nothing
00000002 D __malloc_margin
00000002 D __malloc_heap_start
00000002 D __malloc_heap_end
00000002 B __flp
00000002 B errno
00000002 B __brkval
00000001 b SPIClass::initialized
00000001 B TwoWire::txAddress
00000001 B TwoWire::txBufferLength
00000001 B TwoWire::rxBufferLength
00000001 B TwoWire::txBufferIndex
00000001 B TwoWire::rxBufferIndex
00000001 B TwoWire::transmitting
00000001 B RFM69::TARGETID
00000001 b RFM69::SENDERID
00000001 B RFM69::DATALEN
00000001 B RFM69::_mode
00000001 B RFM69::ACK_REQUESTED
00000001 b RFM69::ACK_RECEIVED
00000001 b RFM69::PAYLOADLEN
00000001 B _ZL22_firmwareUpdateOngoing.lto_priv.93
00000001 D _ZL15_transportToken.lto_priv.105
00000001 b _firmwareRetry
00000001 D _wokeUpByInterrupt
00000001 d _wakeUp2Interrupt
00000001 d _wakeUp1Interrupt
00000001 b twi_txBufferLength
00000001 b twi_txBufferIndex
00000001 B twi_state.lto_priv.95
00000001 B twi_slarw.lto_priv.100
00000001 B twi_sendStop.lto_priv.96
00000001 b twi_rxBufferIndex
00000001 B twi_masterBufferLength.lto_priv.99
00000001 B twi_masterBufferIndex.lto_priv.98
00000001 B twi_inRepStart.lto_priv.101
00000001 B twi_error.lto_priv.97
00000001 b timer0_fract
00000001 B _address

minix:deur02 pragtich$ avr-nm -Crtd --size-sort bin/fw-dbg-ota.elf 
00001352 t global constructors keyed to 65535_0_Deur02.ino.cpp.o.2635
00001062 T transportProcess()
00000790 T stInitUpdate()
00000590 t Sensor::loop(MyMessage*)
00000414 T _sleep(unsigned long, bool, unsigned char, unsigned char, unsigned char, unsigned char)
00000370 t loop
00000368 T RFM69::interruptHandler()
00000338 t setup
00000304 T free
00000298 T malloc
00000274 T main
00000266 T RFM69::sendFrame(unsigned char, void const*, unsigned char, bool, bool)
00000238 T presentation()
00000218 T ChildInt::sendValue(bool)
00000216 T hwSleep(unsigned char, unsigned char, unsigned char, unsigned char, unsigned long)
00000210 t before()
00000198 T RFM69::sendACK(void const*, unsigned char)
00000194 T RFM69::sendWithRetry(unsigned char, void const*, unsigned char, unsigned char, unsigned char)
00000184 T Print::printNumber(unsigned long, unsigned char)
00000172 T SensorInterrupt::onInterrupt()
00000166 T hwPowerDown(unsigned char)
00000162 T RFM69::setMode(unsigned char)
00000158 T RFM69::send(unsigned char, void const*, unsigned char, bool)
00000157 B Serial
00000148 T __vector_16
00000142 T HardwareSerial::write(unsigned char)
00000130 T hwInternalSleep(unsigned long)
00000120 T pinMode
00000118 t presentNode()
00000114 T NodeManager::_onInterrupt_2()
00000114 T NodeManager::_onInterrupt_1()
00000112 T transportRouteMessage(MyMessage&)
00000108 T wait(unsigned long, unsigned char, unsigned char) [clone .constprop.9]
00000106 T MyMessage::set(char const*)
00000102 T SensorInterrupt::onReceive(MyMessage*)
00000102 t transportSendWrite(unsigned char, MyMessage&)
00000100 T __vector_18
00000098 T RFM69::receiveBegin()
00000096 T digitalWrite
00000092 t stUplinkUpdate()
00000090 T ChildInt::print(Print&)
00000088 T transportSwitchSM(transportState_t&)
00000086 T Print::write(unsigned char const*, unsigned int)
00000084 t RFM69::readReg(unsigned char)
00000084 T NodeManager::getSensorWithChild(int)
00000084 T NodeManager::sleepOrWait(long)
00000082 T _callbackTransportReady()
00000082 t turnOffPWM
00000082 T digitalRead
00000080 T RFM69::writeReg(unsigned char, unsigned char)
00000080 T SensorInterrupt::onSetup()
00000080 T HardwareSerial::begin(unsigned long, unsigned char) [clone .constprop.40]
00000080 T wait(unsigned long)
00000078 T __vector_2
00000078 T __vector_1
00000078 T atol
00000076 T __vector_19
00000074 T micros
00000070 t RFM69::canSend()
00000069 B node
00000068 T RFM69::receiveDone()
00000068 T HardwareSerial::flush()
00000068 T receive(MyMessage const&)
00000068 t stParentUpdate()
00000068 T __udivmodsi4
00000066 t HardwareSerial::_tx_udr_empty_irq()
00000064 T RFM69::setHighPower(bool)
00000064 t stFailureUpdate()
00000062 T transportAssignNodeID(unsigned char)
00000062 t stIDTransition()
00000061 B RFM69::DATA
00000060 T Sensor::getChild(int)
00000060 T atoi
00000056 t stFailureTransition()
00000052 T RFM69::setHighPowerRegs(bool)
00000052 T RFM69::setPowerLevel(unsigned char)
00000052 T NodeManager::_sleepBetweenSend()
00000052 t stIDUpdate()
00000050 T build(MyMessage&, unsigned char, unsigned char, unsigned char, unsigned char, bool) [clone .constprop.48]
00000050 t stParentTransition()
00000050 T attachInterrupt
00000048 T present(unsigned char, unsigned char, char const*, bool)
00000046 t _GLOBAL__sub_D__Z6hwInitv
00000044 T RFM69::select()
00000044 t stUplinkTransition()
00000044 T stInitTransition()
00000042 T MyMessage::clear()
00000040 T HardwareSerial::read()
00000040 T __udivmodhi4
00000040 B door
00000040 T __divmodhi4
00000038 T MyMessage::getByte() const [clone .constprop.30]
00000038 T transportTimeInState()
00000036 t stReadyTransition()
00000036 T eeprom_update_byte
00000036 T detachInterrupt
00000033 B _msgTmp
00000033 B _msg
00000032 D vtable for RFM69
00000032 t wakeUp2()
00000032 t wakeUp1()
00000032 T eeprom_update_block
00000032 T eeprom_read_block
00000030 T MyMessage::set(unsigned long)
00000030 T RFM69::unselect()
00000030 T HardwareSerial::availableForWrite()
00000030 T __umulhisi3
00000030 T strncpy
00000028 T HardwareSerial::peek()
00000024 T MyMessage::set(unsigned char)
00000024 T HardwareSerial::available()
00000024 T millis
00000024 T attachInterrupt.part.0.lto_priv.53
00000022 t RFM69::isr0()
00000022 T __muluhisi3
00000022 T __do_global_dtors
00000022 T __do_global_ctors
00000022 T __do_copy_data
00000020 t Serial0_available()
00000020 T serialEventRun()
00000020 t stReadyUpdate()
00000020 T __vector_6
00000020 t digital_pin_to_timer_PGM
00000020 T digital_pin_to_port_PGM
00000020 T digital_pin_to_bit_mask_PGM
00000018 d vtable for HardwareSerial
00000018 T memcpy
00000016 T __do_clear_bss
00000015 B _ZL12_transportSM.lto_priv.55
00000014 D vtable for Sensor
00000014 D vtable for SensorInterrupt
00000014 D vtable for SensorDoor
00000014 t SPIClass::transfer(unsigned char)
00000014 T transportSendRoute(MyMessage&)
00000012 T ChildInt::reset()
00000012 T __tablejump2__
00000012 B _radio
00000010 d vtable for ChildInt
00000010 d vtable for Child
00000010 t port_to_output_PGM
00000010 T port_to_mode_PGM
00000010 t port_to_input_PGM
00000010 T abort
00000006 T Print::availableForWrite()
00000004 B NodeManager::_last_interrupt_2
00000004 B NodeManager::_last_interrupt_1
00000004 d stFailure
00000004 d stUplink
00000004 d stParent
00000004 d stReady
00000004 D _ZL6stInit.lto_priv.56
00000004 d stID
00000004 B _ZL16_transportConfig.lto_priv.51
00000004 B timer0_overflow_count
00000004 B timer0_millis
00000004 D intFunc.lto_priv.52
00000004 T __cxa_pure_virtual
00000002 T Sensor::onReceive(MyMessage*)
00000002 T Sensor::onBefore()
00000002 T Sensor::onSetup()
00000002 T Sensor::onLoop(Child*)
00000002 T Sensor::onInterrupt()
00000002 b RFM69::RSSI
00000002 T RFM69::interruptHook(unsigned char)
00000002 b RFM69::selfPointer
00000002 T Print::flush()
00000002 T Child::sendValue(bool)
00000002 T Child::reset()
00000002 T Child::print(Print&)
00000002 T SensorInterrupt::onLoop(Child*)
00000002 B NodeManager::_last_interrupt_value
00000002 D NodeManager::_last_interrupt_pin
00000002 B _ZL11_coreConfig.lto_priv.54
00000002 B _transportReady_cb
00000002 t nothing
00000002 D __malloc_margin
00000002 D __malloc_heap_start
00000002 D __malloc_heap_end
00000002 B __flp
00000002 B __brkval
00000001 b SPIClass::initialized
00000001 B RFM69::TARGETID
00000001 b RFM69::SENDERID
00000001 B RFM69::DATALEN
00000001 B RFM69::_mode
00000001 B RFM69::ACK_REQUESTED
00000001 b RFM69::ACK_RECEIVED
00000001 b RFM69::PAYLOADLEN
00000001 D _ZL15_transportToken.lto_priv.57
00000001 D _wokeUpByInterrupt
00000001 d _wakeUp2Interrupt
00000001 d _wakeUp1Interrupt
00000001 b timer0_fract
00000001 B _address

minix:deur02 pragtich$ avr-nm -Crtd --size-sort bin/fw-dbg+ota.elf 
00002418 T transportProcess()
00001468 t global constructors keyed to 65535_0_Deur02.ino.cpp.o.2956
00000790 T stInitUpdate()
00000604 T __vector_24
00000590 t Sensor::loop(MyMessage*)
00000502 T _sleep(unsigned long, bool, unsigned char, unsigned char, unsigned char, unsigned char)
00000452 T extEEPROM::write(unsigned long, unsigned char*, unsigned int) [clone .part.1] [clone .constprop.42]
00000370 T loop
00000368 T RFM69::interruptHandler()
00000338 T setup
00000304 T free
00000298 T malloc
00000290 T main
00000266 T RFM69::sendFrame(unsigned char, void const*, unsigned char, bool, bool)
00000238 T presentation()
00000218 T ChildInt::sendValue(bool)
00000216 T hwSleep(unsigned char, unsigned char, unsigned char, unsigned char, unsigned long)
00000210 T before()
00000198 T RFM69::sendACK(void const*, unsigned char)
00000194 T RFM69::sendWithRetry(unsigned char, void const*, unsigned char, unsigned char, unsigned char)
00000184 T TwoWire::endTransmission(unsigned char)
00000184 T Print::printNumber(unsigned long, unsigned char)
00000184 T presentNode()
00000172 T SensorInterrupt::onInterrupt()
00000166 T hwPowerDown(unsigned char)
00000162 T RFM69::setMode(unsigned char)
00000158 T RFM69::send(unsigned char, void const*, unsigned char, bool)
00000157 B Serial
00000148 T __vector_16
00000142 T HardwareSerial::write(unsigned char)
00000130 T hwInternalSleep(unsigned long)
00000120 T pinMode
00000114 T NodeManager::_onInterrupt_2()
00000114 T NodeManager::_onInterrupt_1()
00000112 T transportRouteMessage(MyMessage&)
00000108 T wait(unsigned long, unsigned char, unsigned char) [clone .constprop.10]
00000106 T MyMessage::set(char const*)
00000102 T SensorInterrupt::onReceive(MyMessage*)
00000102 t transportSendWrite(unsigned char, MyMessage&)
00000100 T __vector_18
00000098 T RFM69::receiveBegin()
00000096 T TwoWire::write(unsigned char const*, unsigned int)
00000096 T digitalWrite
00000094 T TwoWire::write(unsigned char)
00000092 t stUplinkUpdate()
00000090 T ChildInt::print(Print&)
00000088 T transportSwitchSM(transportState_t&)
00000086 T Print::write(unsigned char const*, unsigned int)
00000084 t RFM69::readReg(unsigned char)
00000084 T NodeManager::getSensorWithChild(int)
00000084 T NodeManager::sleepOrWait(long)
00000082 t _callbackTransportReady()
00000082 t turnOffPWM
00000082 T digitalRead
00000080 T RFM69::writeReg(unsigned char, unsigned char)
00000080 T SensorInterrupt::onSetup()
00000080 T HardwareSerial::begin(unsigned long, unsigned char) [clone .constprop.63]
00000080 T wait(unsigned long)
00000080 t twi_transmit
00000078 T __vector_2
00000078 T __vector_1
00000078 T atol
00000076 T __vector_19
00000074 T micros
00000070 t RFM69::canSend()
00000069 B node
00000068 T RFM69::receiveDone()
00000068 T HardwareSerial::flush()
00000068 T receive(MyMessage const&)
00000068 t stParentUpdate()
00000068 T __udivmodsi4
00000066 t HardwareSerial::_tx_udr_empty_irq()
00000064 T I2CEeprom::busy() [clone .constprop.45]
00000064 T RFM69::setHighPower(bool)
00000064 t stFailureUpdate()
00000062 T transportAssignNodeID(unsigned char)
00000062 t stIDTransition()
00000061 B RFM69::DATA
00000060 T Sensor::getChild(int)
00000060 T atoi
00000056 t stFailureTransition()
00000052 T RFM69::setHighPowerRegs(bool)
00000052 T RFM69::setPowerLevel(unsigned char)
00000052 T NodeManager::_sleepBetweenSend()
00000052 t stIDUpdate()
00000050 T build(MyMessage&, unsigned char, unsigned char, unsigned char, unsigned char, bool) [clone .constprop.71]
00000050 t stParentTransition()
00000050 T attachInterrupt
00000048 T present(unsigned char, unsigned char, char const*, bool)
00000046 t _GLOBAL__sub_D__Z6hwInitv
00000044 T RFM69::select()
00000044 t stUplinkTransition()
00000044 T stInitTransition()
00000042 T MyMessage::clear()
00000040 T TwoWire::read()
00000040 T HardwareSerial::read()
00000040 T __udivmodhi4
00000040 B door
00000040 T __divmodhi4
00000038 T MyMessage::getByte() const [clone .constprop.32]
00000038 T transportTimeInState()
00000036 t stReadyTransition()
00000036 T eeprom_update_byte
00000036 T detachInterrupt
00000033 B _msgTmp
00000033 B _msg
00000032 D vtable for RFM69
00000032 b TwoWire::txBuffer
00000032 B TwoWire::rxBuffer
00000032 t wakeUp2()
00000032 t wakeUp1()
00000032 b twi_txBuffer
00000032 b twi_rxBuffer
00000032 B twi_masterBuffer.lto_priv.89
00000032 T eeprom_update_block
00000032 T eeprom_read_block
00000030 T MyMessage::set(unsigned long)
00000030 T TwoWire::peek()
00000030 T RFM69::unselect()
00000030 T HardwareSerial::availableForWrite()
00000030 T __umulhisi3
00000030 T strncpy
00000028 T HardwareSerial::peek()
00000026 T memcmp
00000024 T MyMessage::set(unsigned char)
00000024 T HardwareSerial::available()
00000024 T millis
00000024 T attachInterrupt.part.0.lto_priv.77
00000022 t RFM69::isr0()
00000022 T __muluhisi3
00000022 T __do_global_dtors
00000022 T __do_global_ctors
00000022 T __do_copy_data
00000020 t Serial0_available()
00000020 T serialEventRun()
00000020 t stReadyUpdate()
00000020 T __vector_6
00000020 t twi_stop
00000020 t digital_pin_to_timer_PGM
00000020 T digital_pin_to_port_PGM
00000020 T digital_pin_to_bit_mask_PGM
00000018 d vtable for TwoWire
00000018 d vtable for HardwareSerial
00000018 T memcpy
00000016 T TwoWire::available()
00000016 B _flash
00000016 T __do_clear_bss
00000015 B _ZL12_transportSM.lto_priv.79
00000014 D vtable for Sensor
00000014 D vtable for SensorInterrupt
00000014 D vtable for SensorDoor
00000014 t SPIClass::transfer(unsigned char)
00000014 T transportSendRoute(MyMessage&)
00000012 T ChildInt::reset()
00000012 B Wire
00000012 T __tablejump2__
00000012 B _radio
00000010 d vtable for ChildInt
00000010 d vtable for Child
00000010 t port_to_output_PGM
00000010 T port_to_mode_PGM
00000010 t port_to_input_PGM
00000010 T abort
00000008 B _ZL19_nodeFirmwareConfig.lto_priv.81
00000006 T Print::availableForWrite()
00000004 B NodeManager::_last_interrupt_2
00000004 B NodeManager::_last_interrupt_1
00000004 d stFailure
00000004 d stUplink
00000004 d stParent
00000004 d stReady
00000004 D _ZL6stInit.lto_priv.90
00000004 d stID
00000004 b _firmwareLastRequest
00000004 B _ZL16_transportConfig.lto_priv.75
00000004 B timer0_overflow_count
00000004 B timer0_millis
00000004 D intFunc.lto_priv.76
00000004 T __cxa_pure_virtual
00000002 T TwoWire::flush()
00000002 T Sensor::onReceive(MyMessage*)
00000002 T Sensor::onBefore()
00000002 T Sensor::onSetup()
00000002 T Sensor::onLoop(Child*)
00000002 T Sensor::onInterrupt()
00000002 b RFM69::RSSI
00000002 T RFM69::interruptHook(unsigned char)
00000002 b RFM69::selfPointer
00000002 T Print::flush()
00000002 T Child::sendValue(bool)
00000002 T Child::reset()
00000002 T Child::print(Print&)
00000002 T SensorInterrupt::onLoop(Child*)
00000002 B NodeManager::_last_interrupt_value
00000002 D NodeManager::_last_interrupt_pin
00000002 b _firmwareBlock
00000002 B _ZL11_coreConfig.lto_priv.78
00000002 b twi_onSlaveTransmit
00000002 b twi_onSlaveReceive
00000002 B _transportReady_cb
00000002 t nothing
00000002 D __malloc_margin
00000002 D __malloc_heap_start
00000002 D __malloc_heap_end
00000002 B __flp
00000002 B __brkval
00000001 b SPIClass::initialized
00000001 B TwoWire::txAddress
00000001 B TwoWire::txBufferLength
00000001 B TwoWire::rxBufferLength
00000001 B TwoWire::txBufferIndex
00000001 B TwoWire::rxBufferIndex
00000001 B TwoWire::transmitting
00000001 B RFM69::TARGETID
00000001 b RFM69::SENDERID
00000001 B RFM69::DATALEN
00000001 B RFM69::_mode
00000001 B RFM69::ACK_REQUESTED
00000001 b RFM69::ACK_RECEIVED
00000001 b RFM69::PAYLOADLEN
00000001 B _ZL22_firmwareUpdateOngoing.lto_priv.80
00000001 D _ZL15_transportToken.lto_priv.91
00000001 b _firmwareRetry
00000001 D _wokeUpByInterrupt
00000001 d _wakeUp2Interrupt
00000001 d _wakeUp1Interrupt
00000001 b twi_txBufferLength
00000001 b twi_txBufferIndex
00000001 B twi_state.lto_priv.82
00000001 B twi_slarw.lto_priv.87
00000001 B twi_sendStop.lto_priv.83
00000001 b twi_rxBufferIndex
00000001 B twi_masterBufferLength.lto_priv.86
00000001 B twi_masterBufferIndex.lto_priv.85
00000001 B twi_inRepStart.lto_priv.88
00000001 B twi_error.lto_priv.84
00000001 b timer0_fract
00000001 B _address


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
