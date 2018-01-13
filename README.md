# f2mc8dump

![Photo](https://user-images.githubusercontent.com/52712/34909015-cdebd7d4-f84e-11e7-86c4-c4403cf749d8.png)

## Overview

Some Fujitsu F2MC-8L microcontrollers with internal mask ROM also have an
external bus capability (see the [catalog](https://web.archive.org/web/20170514004456/http://www.fujitsu.com/downloads/MICRO/fme/micros/micros_2006.pdf)). A mode pin selects external bus mode and then some GPIO pins are repurposed
as address and data lines, like on the 8051 or AVRs. In external bus mode, an
external ROM is used. There is no access to the internal ROM when external bus
mode is active.

I dumped the internal mask ROM of the several F2MC8-L microcontrollers by
hooking up an external EPROM that writes a dumping program into RAM, then
jumps to RAM. While that is running, I change the mode pin to select the
internal ROM. The external bus is disabled, the internal ROM is enabled, but
the code in RAM keeps running and the internal ROM is dumped out. I've run it
many times on several chips and it has been perfectly reliable.

## Hardware

The hardware setup requires the F2MC-8L microcontroller be removed from the
target device. It is placed on a board with the following other components:

 - 2764 or similar parallel EPROM with dumping firmware
 - 74HCT373 transparent latch
 - 74HCT138 3-8 decoder
 - 5V TTL oscillator (1-8 MHz)
 - 5V regulator, decoupling caps, pull-up resistors, etc.

See the `hardware/` directory for details.

A logic analyzer with 9 probes (8 data lines, 1 strobe line) is required to
capture the output from the dumping hardware.

## Firmware

Assemble the dumping firmware in the `firmware/` directory using the [asf2mc8](https://shop-pdp.net/ashtml/asf2mc.htm)
assembler and burn it into the EPROM.

## Dumping

Build the hardware and power it up in external bus mode (MOD0=Vcc). It will
immediately begin dumping the memory space out P30-P37 with /STROBE on P40
after each byte. It will wrap around memory and keep dumping forever. If you
capture it with a logic analyzer, you'll see the string `EXTERNALROM` from
the dumping firmware.

While power is still applied, change MOD0 to Vss. This selects the internal
ROM mode. The program will continue dumping but it will now be dumping the
internal ROM instead. Capture the output with a logic analyzer and let it
run for a couple minutes so the entire memory space is dumped several times.
Save the capture to a file.

Use the Python script under the `software/` directory to individual ROM images
from the logic analyzer capture.

## Author

[Mike Naberezny](https://github.com/mnaberez)
