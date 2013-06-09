fx2cam
======

Need to grab images from a CMOS sensor over USB using a Cypress FX2LP?  Then boy have I got a deal for you.

This project includes code for the host (typically a Linux PC) and the microcontroller
(Cypress "EZ-USB" FX2LP CY7C68013A/14/15/16A)
It allows you to read and write camera registers over I2C, and stream the image data at up to 24 MB/s using
USB 2.0 high-speed isochronous transfers.  It uses the following open-source projects:

* sdcc - Small Device C Compiler, targeting the 8051 core of the FX2LP
* fx2lib - Library and framework for implementing USB peripherals in the FX2LP
* libusb 1.0 - Host-side C library allowing user-space access to USB devices at a low level

Creator: Henry Hallam, fx2cam@pericynthion.org

License: WTFPL

Support / Discussion: fx2cam@googlegroups.com


Hardware
--------

You need to provide your own.  It should be centered around the CMOS imaging sensor of your choice (or, I guess,
any other source of high-speed parallel data), and a Cypress FX2LP USB microcontroller.

### Important connections:
* Power supplies - If you can't figure that stuff out, this is probably not the project for you. 
* I2C - connect the sensor's I2C bus to the FX2LP.  Don't forget the 1.5k pull-up resistors.  You can add an EEPROM
  on that bus if you'd like it to identify on USB with a PID/VID other than the Cypress default 04B4:8613.
* Imaging sensor reset - make sure there's a way to reset the imaging sensor.
  I just connect its !RESET to the FX2LP's !RESET.
* FX2LP clock, reset, wakeup etc - follow the recommendations in the EZ-USB TRM, or copy the FX2LP development kit
  schematic.
* Imaging sensor input clock - I used the FX2LP's CLKOUT pin to source a 48 MHz clock.
* Imaging sensor output clock - it better be < 30 MHz, < ~12 MHz if you're using more than 8 bits of pixel data.
  Hook it to the FX2LP's IFCLK.
* Parallel pixel data - If 8 bit, connect to FD0..7.  If > 8 bit, connect to FD0..15, tying any unused MSBs to ground
  or you can put the line valid / frame valid signals on them if you like.
* Image sensor's Line Valid (LV) output - connect to FX2LP's SLWR
* Image sensor's Frame Valid (FV) output - connect to FX2LP's PA0 or another GPIO.
* Does your sensor give you HSYNC/HREF and VSYNC/VREF instead of LV and FV?  That's inconvenient.  Maybe you could:
  * Try to use the FX2LP's GPIF.
  * Add some external hardware to count pixels and generate line valid signals (possibly assisted by FX2LP firmware).
  * Just write SLWR to +3V, stream everything to the host and sort it out there.
* FX2LP's SLRD, SLOE and FIFOADR1: GND those suckas
* FX2LP's FIFOADR0:  You can ground it, or if your sensor puts out LV=1 even when FV=0 (vertical blanking interval)
or you need to otherwise suppress writing into the FIFO, you can connect it to a GPIO like PA1 so that firmware can
deselect the Endpoint 2 FIFO (the only big FIFO that we're using operationally)
