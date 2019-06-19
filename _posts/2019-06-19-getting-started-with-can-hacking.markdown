---
layout: post
title:  Getting started with CAN (Car hacking)
date:   2019-06-19 12:14:24 +0200
categories: 
---

There are some great resources for hacking vehicles. The [Car Hacker Handbook](http://opengarages.org/handbook/ebook/) is one of the best ones. It describes in some detail the different protocols, tools and attacks commonly used when doing a penetration test against a vehicle. 

I'm going to briefly describe how I got started when I wanted to learn more about the CAN protocol. First of all I needed some tools to actually be able to connect to a CAN bus. I've been using two cheap tools which are both great and easy to get started with. The first one is CANUSB and is a simple cable using SocketCAN (slcan utility). The other one is PiCAN and requires a Raspberry Pi 2 or 3. 

- [CANUSB](https://www.canusb.com/)
- [PiCAN Hat for Raspberry Pi 2 and 3](https://copperhilltech.com/pican-2-can-interface-for-raspberry-pi-2-3/)

When using the PiCAN you need to setup your Raspberry Pi (running Rasbian) by doing the following:

Add the can0 interface to `/etc/network/interfaces`
```
auto can0
iface can0 inet manual
pre-up /sbin/ip link set can0 type can bitrate 500000 triple-sampling on
up /sbin/ifconfig can0 up
down /sbin/ifconfig can0 down
```

Add the following to `/boot/config.txt`
```
dtparam=spi=on
dtoverlay=mcp2515-can0,oscillator=16000000,interrupt=25
dtoverlay=spi-bcm2835
```

And that's it for PiCAN!

To use the CANUSB it's even simpler. Just install the can-utils package (Ubuntu) and add the device using `slcand`. First, plugin the CANUSB and find which device name it has been given (ex. /dev/ttyUSB0). Then run the following commands. 
```
slcand -o -c -f -s6 /dev/ttyUSB0 can0
sleep 2
ip link set dev can0 up
```

And that's it for CANUSB!


### Resources
- [https://hackaday.com/2013/10/22/can-hacking-the-in-vehicle-network/](https://hackaday.com/2013/10/22/can-hacking-the-in-vehicle-network/)
- [http://opengarages.org/handbook/ebook/](http://opengarages.org/handbook/ebook/)
- [http://illmatics.com/carhacking.html](http://illmatics.com/carhacking.html)
- [http://www.can-wiki.info/doku.php?id=can_higher_layer_protocols:main](http://www.can-wiki.info/doku.php?id=can_higher_layer_protocols:<main></main>)
- [https://github.com/jaredthecoder/awesome-vehicle-security](https://github.com/jaredthecoder/awesome-vehicle-security)