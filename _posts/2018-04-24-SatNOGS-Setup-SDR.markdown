---
layout: post
title:  "SatNOGS - Ground Station, pt. 2 - SDR"
date:   2018-04-24 20:50:42 -0700
categories: satnogs sdr
---
# SDR = Satellite Defined Radio

Getting the SDR set up is pretty dang simple.

**Steps I took**
1. Buy [RTL-SDR dongle][rtl-sdr-amazon]
2. Follow the [RTL-SDR quickstart guide][rtl-sdr-quickstart]
   - Download SDR# from [Airspy][airspy]
   - Windows SDR Software Package (filename: sdrsharp-x86.zip)
   - Extract zip file to C:/[sdr or whatever] _//don't put it in Program Files; it fails when running there_
   - Execute install-rtlsdr.bat
   - Plug in the dongle
   - Run zadig.exe to install the proper driver
   - Open SDRSharp.exe and tune in to [KUPD or your-favorite-radio-station] to verify it works

### References
[ARRL: The National Association of Ameteur Radio][arrl]

[rtl-sdr-amazon]:https://www.amazon.com/RTL-SDR-Blog-RTL2832U-Software-Telescopic/dp/B011HVUEME/ref=sr_1_3?ie=UTF8&qid=1524711436&sr=8-3&keywords=rtl+sdr
[rtl-sdr-quickstart]:https://www.rtl-sdr.com/rtl-sdr-quick-start-guide/
[airspy]:https://airspy.com/download/
[arrl]:http://www.arrl.org
