---
layout: post
title:  "SatNOGS - Ground Station, pt. 1 - Raspberry Pi"
date:   2018-04-22 20:50:42 -0700
categories: satnogs RaspberryPi
---
# Set up a SatNOGS Client on a Raspberry Pi

To set up my Raspberry Pi, I followed the [SatNOGS wiki's Raspberry Pi 3][satnogs-wiki-rpi3]
page, which contains links to the client software in [GitLab][satnogs-rpi3-img],
RPi's [flashing instructions][rpi3-flash] and steps to do the initial config
once the RPi is booted up.  I also referred to the [Raspberry Pi documentation][rpi3-docs].

Here's a breakdown...

**Hardware Needed:**
* Raspberry Pi 3 Model B
* Micro SD Card (32GB max)
* HDMI Monitor
* Power Supply (2+ Amps at 5V; like a phone charger)
* USB Keyboard
* USB Mouse (optional)

**Software Needed:**
* [SatNOGS-specific Raspbian image][satnogs-rpi3-img] (most recent, stable tag)
* [Etcher][etcher-download] (SD Card Writer tool)

**Steps:**
1. [Download][satnogs-rpi3-img] the Raspbian SatNOGS zipped image, info and checksum
2. [Flash][rpi3-flash] the Raspbian-SatNOGS-lite image to the microSD card with [Etcher][etcher-download]
3. Plug in the microSD and the monitor, then plug in the power.
4. Once the RPi boots up, plug in the keyboard and mouse.
5. Log in.  (Defaults for raspbian are user = pi, password = raspberry)
6. `passwd` to change your password
7. `sudo raspi-config` and followed directions on the [satnogs wiki RPi page][satnogs-wiki-rpi3] to configure my device (default is UK/English settings)

[satnogs-wiki-rpi3]:https://wiki.satnogs.org/Raspberry_Pi_3#Raspbian
[satnogs-rpi3-img]:https://gitlab.com/librespacefoundation/satnogs/satnogs-pi-gen/tags
[rpi3-docs]:https://www.raspberrypi.org/documentation/
[rpi3-flash]:https://www.raspberrypi.org/documentation/installation/installing-images/README.md
[etcher-download]:https://etcher.io/
