---
title: Install ESXi from the Onboard Administrator
date: 2011-05-03
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
showComments: true
---

Installing an ESXi server, or any other operative system, using the [c-Class Blade Enclosure](http://h18004.www1.hp.com/products/blades/components/enclosures/c-class/index.html) Onboard Administrator and the server ILO is a very easy and straightforward process. The Onboard Administrator integrates with each blade server iLO, allowing pass-through authentication.

[![](/images/oa.png "Onboard Administrator")](/images/oa.png)

From the OA main screen go to Device Bays and select the blade server where you want to install the ESXi on, from there enter the iLO section.

[![](/images/device_bay1.png "Device Bay selection")](/images/device_bay1.png)

In the iLO area of the server launch a console applet; as you can see in the below screenshot there are several options, choose the one the better suits your environment.

[![](/images/ilo.png "iLO applets")](/images/ilo.png)

I personally choose **Remote Console**. When the console is up you'll see a menu, open the Virtual Drives menu. Here you can map a physical drive from your PC/Laptop, an USB Key or an ISO image.

[![](/images/virtual_device.png "Virtual Devices")](/images/virtual_device.png)

With the media mapped to the blade server reset the server from the Power Switch menu and launch the ESXi installation like in any other server. By default it boot from the DVD if not from the boot screen press F11 to launch the Boot Menu.

[![](/images/boot_screen.png "Proliant boot screen")](/images/boot_screen.png)

If everything goes as expected after a few minutes you will have a new ESXi server. Just remember to disconnect the virtual drive from the blade once the ESXi is installed or it will boot again from the ISO image after the reboot of the server.

[![](/images/esxi.png "ESXi")](/images/esxi.png)

Finally is important to point out that the install process can be launched simultaneously to several servers using the same ISO image.

Juanma.
