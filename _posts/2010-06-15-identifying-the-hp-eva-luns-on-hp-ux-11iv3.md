---
layout: post
title: Identifying the HP EVA LUNs on HP-UX 11iv3
date: 2010-06-15 09:40:17.000000000 +02:00
type: post
published: true
status: publish
categories:
- HP-UX
- Storage
- Sysadmin
tags:
- 11iv3
- Command View
- EVA
- HP-UX
- scsimgr
- Storage
author: juan_manuel_rey
---

Yesterday's post about CLARiiON reminded me a similar issue I observed when the storage array is an HP EVA. If you ask for the disk serial number with `scsimgr` you always get the same number, in fact this number is the serial of the HSV controller.

The key to match your disk in the HP-UX host with the LUN provided by the EVA arrays is the *wwid* attribute of the disk.

{% highlight text %}
root@ignite:/ # scsimgr get_attr -D /dev/rdisk/disk10 -a wwid

        SCSI ATTRIBUTES FOR LUN : /dev/rdisk/disk10

name = wwid
current = 0x600508b40006cb700000600008bb0000
default =
saved =

root@ignite:/ #
{% endhighlight %}

If you look for this value in Command View will see that is the same as the World Wide LUN Name and the UUID.

[![](/images/eva_lun-id.jpg "EVA_LUN-ID")]({{site.url}}/images/eva_lun-id.jpg)

### **UPDATE**

Thanks to my friend Jean and to Greg who reminded me that like Greg said in his comment is much easier to match the Word Wide LUN Name with the `evainfo` tool. Thanks to both of you :-)

{% highlight text %}
root@hpux-server # evainfo -aP

Devicefile                      Array                   WWNN                            Capacity        Controller/Port/Mode
/dev/rdisk/disk20       5001-4380-04C7-2D90 6005-08B4-000F-3EED-0000-5000-003A-0000      204800MB       Ctl-A/FP-2/Optimized
/dev/rdisk/disk21       5001-4380-04C7-2D90 6005-08B4-000F-3EED-0000-5000-0042-0000      204800MB       Ctl-A/FP-1/Optimized
/dev/rdisk/disk22       5001-4380-04C7-2D90 6005-08B4-000F-3EED-0000-5000-004A-0000       20480MB       Ctl-A/FP-1/Optimized
/dev/rdisk/disk23       5001-4380-04C7-2D90 6005-08B4-000F-3EED-0000-5000-004E-0000       71680MB       Ctl-A/FP-2/Optimized
/dev/rdisk/disk24       5001-4380-04C7-2D90 6005-08B4-000F-3EED-0000-5000-0052-0000       10240MB       Ctl-A/FP-1/Optimized
/dev/rdisk/disk25       5001-4380-04C7-2D90 6005-08B4-000F-3EED-0000-5000-0056-0000       10240MB       Ctl-A/FP-1/Optimized
/dev/rdisk/disk26       5001-4380-04C7-2D90 6005-08B4-000F-3EED-0000-5000-005A-0000       20480MB       Ctl-A/FP-1/Optimized
/dev/rdisk/disk27       5001-4380-04C7-2D90 6005-08B4-000F-3EED-0000-5000-005E-0000      245760MB       Ctl-A/FP-1/Optimized
{% endhighlight %}

Where can I get EVAinfo? Like Greg said EVAinfo is distributed on the HP StorageWorks Storage System Scripting Utility CD (SSSU) since 8.0 version. <span style="text-decoration:line-through;">Unfortunately I couldn't find, yet, a public download URL but the CD is distributed with the hardware so if you own an EVA is probably you already have the media.</span>

Thanks to Jean, man it seems that I owe you more than a couple of beers ;-D, here it is the URL of the CD. You will find the EVAinfo utility inside the **HP StorageWorks Command View SSSU v9.2 software** ISO.

<https://h20392.www2.hp.com/portal/swdepot/displayProductInfo.do?productNumber=CommandViewEVA9.2>

Juanma.
