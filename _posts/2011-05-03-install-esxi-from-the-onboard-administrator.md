---
title: Install ESXi from the Onboard Administrator
date: 2011-05-03
type: postclasses: wide
published: true
status: publish
categories:
- HP
- Virtualization
- VMware
tags:
- ESXi
- HP
- HP BladeSystem
- HP Onboard Administrator
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

Installing an ESXi server, or any other operative system, using the [c-Class Blade Enclosure](http://h18004.www1.hp.com/products/blades/components/enclosures/c-class/index.html) Onboard Administrator and the server ILO is a very easy and straightforward process. The Onboard Administrator integrates with each blade server iLO, allowing pass-through authentication.

[![](/assets/images/oa.png "Onboard Administrator")]({{site.url}}/assets/images/oa.png)

From the OA main screen go to Device Bays and select the blade server where you want to install the ESXi on, from there enter the iLO section.

[![](/assets/images/device_bay1.png "Device Bay selection")]({{site.url}}/assets/images/device_bay1.png)

In the iLO area of the server launch a console applet; as you can see in the below screenshot there are several options, choose the one the better suits your environment.

[![](/assets/images/ilo.png "iLO applets")]({{site.url}}/assets/images/ilo.png)

I personally choose **Remote Console**. When the console is up you'll see a menu, open the Virtual Drives menu. Here you can map a physical drive from your PC/Laptop, an USB Key or an ISO image.

[![](/assets/images/virtual_device.png "Virtual Devices")]({{site.url}}/assets/images/virtual_device.png)

With the media mapped to the blade server reset the server from the Power Switch menu and launch the ESXi installation like in any other server. By default it boot from the DVD if not from the boot screen press F11 to launch the Boot Menu.

[![](/assets/images/boot_screen.png "Proliant boot screen")]({{site.url}}/assets/images/boot_screen.png)

If everything goes as expected after a few minutes you will have a new ESXi server. Just remember to disconnect the virtual drive from the blade once the ESXi is installed or it will boot again from the ISO image after the reboot of the server.

[![](/assets/images/esxi.png "ESXi")]({{site.url}}/assets/images/esxi.png)

Finally is important to point out that the install process can be launched simultaneously to several servers using the same ISO image.

Juanma.
