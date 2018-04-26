---
layout: post
title:  "SatNOGS - Ground Station, pt. 3 - Antenna"
date:   2018-04-25 19:50:42 -0700
categories: satnogs antenna
---
# Build an Antenna for the Ground Station

I know nothink!  I have no prior radio knowledge, but I've already taken care
of the easy stuff - the rtl-sdr and the raspberry pi - so, it's time to hunker
down and learn some things.

### Notes
**Which Antenna?**
- Most SatNOGS make use of these types of antenna: yagi and helical
  - Helical polarization is circular
  - Yagi polarization is horizontal or vertical
  - Crossed-Yagi polarization is circular (and horizontal/vertical)
- Satellite transmissions can be any of these polarities; hence the different antenna types
- Signal loss when antenna/satellite polarities do not match:
  - Yagis lose about 20dB of signal
  - Helicals lose 3dB
  - Cross-Yagi loss is 3dB too
- Helicals for 435MHz are a more reasonable size than a comprable Cross-Yagi
- [References = SatNOGS members:  planetsofa, g7kse, monroe, blackbird
([see community][satnogs-community])]

**.:. I will build a Helical 435**

### The plan
1. Find and Review Assembly instructions
The [SatNOGS Assembly page][satnogs-assembly] provides _really nice_ step-by-step instructions with pictures.  
2. Print [design files][satnogs-gitlab-helical-v5] for the Helical Antenna **v5**
3. Source the parts
   - 1 Pcs Aluminium (Square tubes profile 20x20mm) = 1600mm
   - 2 Pcs Aluminium (Symmetrical L profiles 15x15mm or 20x20mm) = 680mm
   - 2 Pcs Aluminium (Symmetrical L profiles 15x15mm or 20x20mm) = 420mm
   - 1 pc at least of 70Χ70cm good quality grid mesh of 2mm with at 1in (25,4 mm) spaces.
   - 11 pcs Acetal rods 8mm wide and 140mm long
   - 22 Nylon nuts Μ8
   - 1 N-Type connector
   - Brass wire 3mm approx. 6 m
   - Brass sheet 0.3 mm thick approx 20Χ10cm
4. Take instructions, plans and parts into [HeatSync Labs][heatsync]
(local maker space) and see if they can help me.  ...I want to do it all
myself, but I don't have the tools.
5. Test it.

### Status
Step 2.

### The build

### References

### Books
- [ARRL's antenna book][aarl-antenna-book]
- [Introduction to Satellite Communication by Elbert (Free PDF)][elbert-download]

[aarl-antenna-book]:https://www.arrl.org/shop/ARRL-Antenna-Book-23rd-Softcover-Edition/
[elbert-download]:https://epdf.tips/introduction-to-satellite-communication-artech-house-space-applications.html
[heatsync]:http://www.heatsynclabs.org
[satnogs-assembly]:https://ohai.satnogs.org/group/hardware/
[satnogs-community]:https://community.libre.space/t/what-anteena-to-prefer-helical-or-yagi-and-why/533/7
[satnogs-gitlab-helical-v5]:https://gitlab.com/librespacefoundation/satnogs/satnogs-antennas/tree/master/Helical/UHF-435-8-version_5
