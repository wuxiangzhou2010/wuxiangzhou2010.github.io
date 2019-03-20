---
layout: post
title: "Gigabyte z370n-wifi My first hackintosh build"
date: 2016-05-08 12:25 +0800
categories: tools
published: true
---
# z370n-wifi-hackintosh

## Components

| Component   | Model                         |
| ----------- | ----------------------------- |
| CPU         | Intel i3 8100                 |
| Motherboard | Giga-z370n-wifi itx  Bios: f4 |
| RAM         | Crucial DDR4 2666 8G          |
| Case        | In Win Chopin                 |
| Cooler      | Noctua NH-L9I                 |
| SSD         | Samsung 840 EVO 120G          |

update:

1. 20190317  Samsumg pm981 is not compatible with macos 10.14.2, will reboot. no easy solution found yet.

## Kexs

### Required

- [Lilu](https://github.com/acidanthera/Lilu/releases)

- [FakeSMC](https://bitbucket.org/RehabMan/os-x-fakesmc-kozlek/downloads)

### Audio

- [AppleALC](https://github.com/acidanthera/AppleALC/releases/)
- [CodecCommander](https://bitbucket.org/RehabMan/os-x-eapd-codec-commander/downloads/)

### Video

- [WhateverGreen](https://github.com/acidanthera/WhateverGreen/releases)

### Network

[IntelMausiEthernet](https://bitbucket.org/RehabMan/os-x-intel-network/downloads)

## Todo

1. case
  
    Change the case to a larger one, say Inwin 301 matx case. This will make a lot room for other staff like graphic card.

2. Graphic card

    I found amd card, like rx580 work out of the box, As my girl friend want to do photo work like PS and AE, maybe I need to add a graphic card.

3. wifi
  
    I Plan to use an Apple part to get wifi work out of the box. I do a lot of search and found that `bcm94360cs2` with m.2 adapter works fine. However the Inwin itx case is two small to fit the adapter. A larger case is needed.

4. Magic trackpad

    I heard a lot that Apple's trackpad is a must for real Macos experience.

## Refer

- z370 installation guide:

  https://hackintosher.com/forums/thread/success-gigabyte-z370n-wifi-i5-8400-nvme-970-evo-mojave-10-14-1-vanilla-hackintosh-deluxe-build.704/

- z370 installation guide

    https://hackintosher.com/builds/gigabyte-z370n-wifi-itx-hackintosh-guide-4k-htpc-build/

- clover configuration

    https://mackie100projects.altervista.org/clover-configurator-how-to-use/

- make a installer
    
    https://hackintosher.com/guides/how-to-make-a-macos-10-14-mojave-flash-drive-installer/

- download kexts

    https://hackintosher.com/downloads/kexts/

- fix wifi card
  
  https://www.tonymacx86.com/threads/...card-into-a-ga-z370n-wifi-motherboard.259300/
- hackintosh-vanilla-desktop-guide

    https://hackintosh.gitbook.io/-r-hackintosh-vanilla-desktop-guide/gathering-kexts

- onedrive kext download

    https://onedrive.live.com/?authkey=%21APjCyRpzoAKp4xs&id=FE4038DA929BFB23%21455036&cid=FE4038DA929BFB23