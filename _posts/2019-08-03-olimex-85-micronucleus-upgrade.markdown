---
layout: post
title:  "Upgrading OLIMEXINU-85 to latest micronucleus"
date:   2019-08-03 09:34:54 +0200
categories: electronics attiny85
---
So I'm starting a project to make next years conference badge for [Securityfest](www.securityfest.com). I wanted something people could solder for them selves and have started looking into the [Attiny85-20PU](https://www.mouser.se/ProductDetail/Microchip-Technology-Atmel/ATTINY85-20PU?qs=8jWQYweyg6NCiiaOb5GI9Q==) microcontroller. 

### Features
- 8 pins
- 8kB flash
- 20MHz max freq
- 8 bit
- 512 B ROM
- 6 I/O
- Operating voltage 2.7 - 5.5V

### Introduction
It's a small and cheap microcontroller with 6 I/O pins, GND and VCC. 8 pins in total. It's easy to solder, a lot of online material and fairly easy to get started with. You could use an Arduino UNO as a ISP programmer or use a dedicated programmer like SparkFun's [Tiny AVR Programmer](https://www.sparkfun.com/products/11801). 

I bought ten of these small microcontrollers and an [OLIMEXINO-85-KIT](https://www.olimex.com/Products/Soldering-Kits/OLIMEXINO-85-KIT/open-source-hardware). You could be a assembled one, but I want to practice my soldering skills so that's why I went with the kit. The OLIMEXINO-85 comes with a attiny85 pre-programmed with Bluebie's [fork](https://github.com/Bluebie/micronucleus) of the micronucleus bootloader. An old version which haven't been maintained for six years. So the first thing I did was to upgrade it to the latest release of the official [micronucleus](https://github.com/micronucleus/micronucleus) bootloader. To flash a new bootloader to the attiny85, you need a In-System-Programmer (ISP). I used my Arduino Uno as an ISP programmer. 

### Before you start
Follow [this](http://digistump.com/wiki/digispark/tutorials/connecting) official guide and make sure everything works before you continue with the upgrade. 

### Arduino Uno as ISP programmer
On the Arduino Uno:
- Open the Arduino IDE (1.8.9 in this case)
- Load the example "ArduinoISP"
- Note the baudrate in the sketch (19200 in my case) (`#define BAUDRATE	19200`)
- Make sure you have the board set to Arduino Uno (Under tools->boards)
- Make sure you have the programmer set to AVRISP mkII (tools->programmer)
- Make sure you use the correct port (tools->port). 
- Upload the sketch!

Now, find a 10µF capacitor and place it between GND and RESET on your Arduino Uno. Long leg on RESET, short (The GND marked leg) on GND. This is because we don't want the Arduino to reset after we load the sketch to the attiny85. 

Using a breadboard, connect pin 11 (MOSI), 12 (MISO), 13 (SCK) on the Arduino Uno to your Attiny85 on pins PB0/5/#0 (MOSI), PB1/6/#1 (MISO), PB2/7/#2 (MOSI). Connect pin 10 on the Arduino to PB5/1/RESET on your Attiny85. Connect GND and VCC (3.3V or 5V, I've used both and both work fine). 

### Flashing micronucleus
I used `avrdude` to upload the bootloader. If you're on Ubuntu, you can download it using APT. It could also be a good idea to install avr-libc if you want to compile some code to run on your attiny85. 
```
root@hacktop$ apt install avrdude avr-libc
``` 
Clone the repository and enter the releases directory.
```
user@hacktop$ https://github.com/micronucleus/micronucleus.git
user@hacktop$ cd micronucleus/firmware/releases
```
Now we want to flash the attiny85 using our Arduino Uno with the latest micronucleus bootloader. NOTE! I'm new to Attiny85 and setting the fuses without knowing what you're doing is dangerous. I didn't know what I was doing most of the time. You can come a long way using this [calculator](http://eleccelerator.com/fusecalc/fusecalc.php?chip=attiny85). I got the following command from [this](http://dumbpcs.blogspot.com/2015/11/flashing-your-attiny85-with.html) guide.  

```
root@hacktop$ avrdude -c arduino -p attiny85 -P /dev/ttyACM3 -b 19200 -U flash:w:t85_default.hex:i -U lfuse:w:0xe1:m -U hfuse:w:0xdd:m -U efuse:w:0xfe:m
```

If something went wrong, check your connections or permissions (run as root if it doesn't work the first time). If you have any questions or comments hit me up on [Twitter](https://twitter.com/bewniac).