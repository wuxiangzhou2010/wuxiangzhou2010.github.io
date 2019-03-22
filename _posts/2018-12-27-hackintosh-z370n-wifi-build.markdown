---
layout: post
title: "z370n-wifi hackintosh build"
date: 2018-12-27 12:00 +0800
categories: hackintosh
published: true
---

This is my first Hackintosh build, before I actually started to buy component, I had done a lot of search  mostly on [`tonymacx68`](http://www.tonymacx86.com) and [`hackintosher`](https://hackintosher.com/). This build is mainly for everday use and sometimes a little code development.

 **Components list**

| Component   | Model                         | note          |
| ----------- | ----------------------------- | ------------- |
| CPU         | Intel i3 8100                 | Taobao 659RMB |
| Motherboard | Giga-z370n-wifi itx  Bios: f4 | Xianyu 900RMB |
| RAM         | Crucial DDR4 2666 8G          | Xianyu 265RMB |
| Case        | In Win Chopin                 | Xianyu 400RMB |
| Cooler      | Noctua NH-L9I                 | Taobao 259RMB |
| SSD         | Samsung 840 EVO 120G          | already owned |
| screen      | Dell E2214hv                  | already owned |

The main installation process is as follows

1. Use a MacOS to download MacOS image from Mac app store
2. Format a USB and flash the OS iamge to USB 
3. Mount EFI folder and corret EFI (using clover bootloader/clover configurator)
4. Reboot and set proper BIOS options, save your modification.
5. Boot from USB installer and install MacOS on a SSD
6. Put the same EFI folder on the new MacOS

### Kexs

#### Required

- [Lilu](https://github.com/acidanthera/Lilu/releases)

- [FakeSMC](https://bitbucket.org/RehabMan/os-x-fakesmc-kozlek/downloads)

#### Audio

- [AppleALC](https://github.com/acidanthera/AppleALC/releases/)
- [CodecCommander](https://bitbucket.org/RehabMan/os-x-eapd-codec-commander/downloads/)

#### Video

- [WhateverGreen](https://github.com/acidanthera/WhateverGreen/releases)

#### Network

- IntelMausiEthernet](https://bitbucket.org/RehabMan/os-x-intel-network/downloads)

### Todo

1. case
  
    Change the case to a larger one, say Inwin 301 matx case. This will make a lot room for other staff like graphic card.

2. Graphic card

    I found amd card, like rx580 work out of the box, As my girl friend want to do photo work like PS and AE, maybe I need to add a graphic card.

3. wifi
  
    I Plan to use an Apple part to get wifi work out of the box. I do a lot of search and found that `bcm94360cs2` with m.2 adapter works fine. However the Inwin itx case is two small to fit the adapter. A larger case is needed.

4. Magic trackpad

    I heard a lot that Apple's trackpad is a must for real Macos experience.

### Reference

- [success-gigabyte-z370n-wifi-i5-8400-nvme-970-evo-mojave-10-14-1-vanilla-hackintosh-deluxe-build](https://hackintosher.com/forums/thread/success-gigabyte-z370n-wifi-i5-8400-nvme-970-evo-mojave-10-14-1-vanilla-hackintosh-deluxe-build.704/)
- [gigabyte-z370n-wifi-itx-hackintosh-guide-4k-htpc-build](https://hackintosher.com/builds/gigabyte-z370n-wifi-itx-hackintosh-guide-4k-htpc-build/)
- [clover-configurator-how-to-use](https://mackie100projects.altervista.org/clover-configurator-how-to-use/)
- [how-to-make-a-macos-10-14-mojave-flash-drive-installer](https://hackintosher.com/guides/how-to-make-a-macos-10-14-mojave-flash-drive-installer/)
- [fix wifi card](https://www.tonymacx86.com/threads/...card-into-a-ga-z370n-wifi-motherboard.259300/)
- [hackintosh-vanilla-desktop-guide](https://hackintosh.gitbook.io/-r-hackintosh-vanilla-desktop-guide/gathering-kexts)
- [download kexts](https://hackintosher.com/downloads/kexts/)
- [onedrive kext download](https://onedrive.live.com/?authkey=%21APjCyRpzoAKp4xs&id=FE4038DA929BFB23%21455036&cid=FE4038DA929BFB23)



update:

1. 20190317:  Samsumg pm981(305RMB from Taobao) is not compatible with Macos 10.14.2, will reboot. no easy solution found yet.
