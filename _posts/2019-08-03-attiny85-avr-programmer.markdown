---
layout: post
title:  "OLIMEXINU-85 as an attiny85 ISP"
date:   2019-08-03 10:34:54 +0200
categories: electronics attiny85
---
I just wrote a short blogpost about upgrading OLIMEXINO-85 with the latest micronucleus bootloader. Another thing I did was to make the OLIMEXINO-85 a ISP programmer. This is really simple. You just need to setup your Arduino Uno as an ISP programmer (or use another ISP programmer), follow the instructions from marsairforce's [fork](https://github.com/marsairforce/vusbtiny) of vusbtiny and solder the RST_E bridge on the back of the OLIMEXINO-85 (check the [manual](https://www.olimex.com/Products/Duino/AVR/OLIMEXINO-85-ASM/resources/OLIMEXINO-85_manual.pdf) for details). 

You need to note one thing, when you do this you can't reprogram your attiny85 with your Arduino anymore. You need an High Voltage programmer to do this. 

If something went wrong, check your connections or permissions (run as root if it doesn't work the first time). If you have any questions or comments hit me up on [Twitter](https://twitter.com/bewniac).
