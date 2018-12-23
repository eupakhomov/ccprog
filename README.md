# ccprog
Chipcon CC1110 GPIO-based (bitbang) programmer

Used to program cc1110 devices over GPIO lines on operating systems and boards that support libmraa, like the [Intel Edison](http://www.intel.com/content/www/us/en/do-it-yourself/edison.html) or [Raspberry Pi](https://www.raspberrypi.org/).

## Building

`make`

## Usage

```
Usage: ./ccprog command

 Commands supported: erase reset write halt resume status

 Command line options:
   -p DC,DD,RESET              specify mraa pins for debugging cc chip:
```

## Example

`./ccprog write uart1_alt2_RILEYLINK_US_STDLOC.hex`

### Wiring for Edison

The edison natively uses 1.8V logic levels.  The CC1110 requires 3.3V logic levels, so you will need something like the [Sparkfun GPIO Block](https://www.sparkfun.com/products/13038) to shift the levels up. 

### Why this branch

This branch addresses the issue when C1110 chip integrated on Explorer Hat board (https://www.enhancedradio.com/products/900mhz-explorer-hat) hangs with both red and green LEDs on.
In this case when original ccprog tries to verify chip id before any operation the check doesn't work and 0x00 returned as a chip id, therefor `This code is only tested on CC1110. Unsupported chip id = 0x00` error.
To deal with it this branch removes chip identity check (please check that you have CC1110 chip before using it) so you can reflash the chip.
It also provides possibility to request a chip status and call resume in case the chip processor is halted.

Recommended actions if chip is halted:

Install ccprog tools on your rig: cd ~/src; git clone https://github.com/eupakhomov/ccprog.git
Build (compile) ccprog so you can run it: cd ccprog; make ccprog
Make sure you’ve installed MRAA (https://github.com/intel-iot-devkit/mraa)

Check chip status:

./ccprog -p 16,18,7 status

E.g. you have 178 (dec) as a status which is:

1011 0010 (binary) = 0x80 + 0x20 + 0x10 + 0x02 (hex)

It means chip was flashed succesfully, CPU is not idle, CPU is halted and oscillator is stable (see list below).

* 0x80 - CHIP_ERASE_DONE
* 0x40 - PCON_IDLE
* 0x20 - CPU_HALTED
* 0x10 - POWER_MODE_0
* 0x08 - HALT_STATUS
* 0x04 - DEBUG_LOCKED
* 0x02 - OSCILLATOR_STABLE
* 0x01 - STACK_OVERFLOW

Flash the radio chip:

wget https://github.com/EnhancedRadioDevices/subg_rfspy/releases/download/v0.8-explorer/spi1_alt2_EDISON_EXPLORER_US_STDLOC.hex
./ccprog -p 16,18,7 reset
./ccprog -p 16,18,7 erase
./ccprog -p 16,18,7 write spi1_alt2_EDISON_EXPLORER_US_STDLOC.hex

In case CPU is halted: call resume one or several times until LEDs start blinking:

./ccprog -p 16,18,7 resume

Reboot rig.