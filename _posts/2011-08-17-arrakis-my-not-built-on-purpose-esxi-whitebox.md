---
title: Arrakis, my not built on purpose ESXi whitebox
date: 2011-08-17
type: post
classes: wide
published: true
status: publish
categories:
- Virtualization
- VMware
tags:
- ESXi
- homelab
- Virtualization
- VMware
author: juan_manuel_rey
comments: true
---

If you are willing to see a so great whitebox like Phil Jaenke's ([@RootWyrm](http://twitter.com/RootWyrm)) [BabyDragon](http://rootwyrm.us.to/2010/08/meet-my-esxi-server-the-babydragon/) I'm sorry to say that you'll be terribly disappointed because unlike Phil's beauty mine wasn't built on purpose.

I've been running all my labs within VMware Workstation on my custom made workstation, which by the way was running Windows 7 (64-bit). But recently I decided that it was time to move to a more reliable solution so I converted my Windows 7 system in an ESXi server.

Surprisingly when I installed ESXi 4.1 Update 1 everything was properly recognized so I thought it could be of help to post the hardware configuration for other vGeeks out there that could be looking for working hardware components for their homelabs.

-   Processor: Intel Core 2 Quad Q9550. Supports FT!
-   Memory: 8GB
-   Motherboard: [Asrock Penryn 1600SLI-110dB](http://www.asrock.com/mb/overview.asp?Model=Penryn1600SLI-110dB)
-   Nic: Embedded nVidia NForce Network Controller. Supported under the forcedeth driver

```
~ # ethtool -i vmnic0
driver: forcedeth
version: 0.61.0.1-1vmw
firmware-version:
bus-info: 0000:00:14.0
~ #
```

-   SATA controller: nVidia MCP51 SATA Controller.

```
~ # vmkload_mod -s sata_nv
vmkload_mod module information
 input file: /usr/lib/vmware/vmkmod/sata_nv.o
 Version: Version 2.0.0.1-1vmw, Build: 348481, Interface: ddi_9_1 Built on: Jan 12 2011
 License: GPL
 Required name-spaces:
  com.vmware.vmkapi@v1_1_0_0
 Parameters:
  heap_max: int
    Maximum attainable heap size for the driver.
  heap_initial: int
    Initial heap size allocated for the driver.
~ #
```

-   HDD1: 1 x 120GB SATA 7200RPM Seagate ST3120026AS.
-   HDD2: 1 x 1TB SATA 7200RPM Seagate ST31000528AS.

Finally here it is a screenshot of the vSphere Client connected to the vCenter VM and showing the summary of the host.

[![](/assets/images/arrakis.png "Arrakis")]({{site.url}}/assets/images/arrakis.png)

The other components of my homelab are a QNAP TS-219P NAS and an HP ProCurve 1810G-8 switch. I also have plans to add two more NICs and a SSD to the server as soon as possible and of course to build a new whitebox.

Juanma.
