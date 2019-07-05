---
layout: post
title:  Getting started with CAN (Car hacking)
date:   2019-06-19 12:14:24 +0200
categories: 
---

There are some great resources for hacking vehicles. The [Car Hacker Handbook](http://opengarages.org/handbook/ebook/) is one of the best ones. It describes in some detail the different protocols, tools and attacks commonly used when doing a penetration test against a vehicle. 

# Using hardware

I'm going to briefly describe how I got started when I wanted to learn more about the CAN protocol. First of all I needed some tools to actually be able to connect to a CAN bus. I've been using two cheap hardware tools which are both great and easy to get started with. The first one is CANUSB and is a simple cable making use of SocketCAN (slcan utility in linux). The other one is PiCAN and requires a Raspberry Pi 2 or 3. 

- [CANUSB](https://www.canusb.com/)
- [PiCAN Hat for Raspberry Pi 2 and 3](https://copperhilltech.com/pican-2-can-interface-for-raspberry-pi-2-3/)

First there's some things you need to do on the hardware, which requires soldering. If you want the DB9 connector to connect directly to the CANUSB, you need to bridge the pads 2 and 3 on SJ1, SJ2 and SJ3 on the PiCAN HAT. You should also solder JP3 which will add a 120 Ohms resistor to the CAN-bus for termination (if you don't have a termination resistor anywhere else on your setup), without it you won't be able to see any CAN messages at all. Since both the PiCAN HAT and the CANUSB are male connectors, you should get your hands on a [female-to-female DB9 plug](https://www.amazon.com/db9-female/s?k=db9+female+to+female). 

To setup the PiCAN hat on your RPi2/3 you need to make some configuration. The following setup is from a RPi3B+ running Rasbian. 

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

Reboot, and then the RPi is ready to go! Just connect the CAN_H and CAN_L wires to your CAN-bus and you should be able to see some traffic using the `candump` command (part of can-utils).

To use the CANUSB it's even simpler. Just install the can-utils package (Ubuntu) and add the device using `slcand`. First, plugin the CANUSB and find which device name it has been given (ex. /dev/ttyUSB0). Then run the following commands. 
```
slcand -o -c -f -s6 /dev/ttyUSB0 can0
sleep 2
ip link set dev can0 up
```
The `-s6` depends on the bitrate. The CAN-bus in this example is setup for 500k, but that value depends on the CAN-bus. 

# Virtual
You don't really need all the hardware to start playing around with the CAN protocol. You could create virtual interfaces using the vcan kernel module in linux.
```
modprobe vcan
ip link add name vcan0 type vcan
ip link set dev vcan0 up
```

Along with the can-utils package, you can run `cangen vcan0` to generate some random CAN frames on the virtual CAN interface.

### Resources
- [https://hackaday.com/2013/10/22/can-hacking-the-in-vehicle-network/](https://hackaday.com/2013/10/22/can-hacking-the-in-vehicle-network/)
- [http://opengarages.org/handbook/ebook/](http://opengarages.org/handbook/ebook/)
- [http://illmatics.com/carhacking.html](http://illmatics.com/carhacking.html)
- [http://www.can-wiki.info/doku.php?id=can_higher_layer_protocols:main](http://www.can-wiki.info/doku.php?id=can_higher_layer_protocols:<main></main>)
- [https://github.com/jaredthecoder/awesome-vehicle-security](https://github.com/jaredthecoder/awesome-vehicle-security)
- [https://scapy.readthedocs.io/en/latest/layers/automotive.html](https://scapy.readthedocs.io/en/latest/layers/automotive.html)