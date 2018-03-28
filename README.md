# f2mc8dump

![Photo](https://user-images.githubusercontent.com/52712/34909015-cdebd7d4-f84e-11e7-86c4-c4403cf749d8.png)

## Overview

Some Fujitsu F2MC-8L microcontrollers have an
external bus capability (see the [catalog](https://web.archive.org/web/20170514004456/http://www.fujitsu.com/downloads/MICRO/fme/micros/micros_2006.pdf)).
I found that the external bus can be used to dump the contents of the
internal ROM, which is not normally accessible.

A mode pin enables the external bus.  In this mode, the internal ROM is disabled and its address space is redirected
to the external bus.  I dumped the internal ROM of the several F2MC-8L microcontrollers by hooking up an external EPROM that wrote a dumping program into RAM, then jumped to RAM.  While code was executing from RAM, I changed the mode pin.  The external bus was disabled, the internal ROM was enabled, but code kept executing from RAM and dumped out the contents of the internal ROM.

## Tested

I developed this method to extract the firmware from the [Volkswagen Premium IV](https://github.com/mnaberez/vwradio) car radios made by Clarion.  These radios contain the following microcontrollers:

 - MB89623R
 - MB89625R
 - MB89677AR

I successfully extracted the internal ROM code from all of them.  I've also tried samples of the above chips from other products and was able to extract the code from them as well.

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

Assemble the dumping firmware in the `firmware/` directory using the [asf2mc8](http://shop-pdp.net/ashtml/asf2mc.htm)
assembler and burn it into the EPROM.

## Dump

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

Use the Python script under the `host/` directory to split individual
ROM images from the logic analyzer capture.

## Disassemble

After a binary has been obtained, use [f2mc8dasm](https://github.com/mnaberez/f2mc8dasm) to disassemble it.

## Author

[Mike Naberezny](https://github.com/mnaberez)
