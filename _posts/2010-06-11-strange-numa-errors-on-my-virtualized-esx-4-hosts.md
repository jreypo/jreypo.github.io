---
layout: post
title: Strange NUMA errors on my virtualized ESX 4 hosts
date: 2010-06-11
type: post
published: true
status: publish
categories:
- Virtualization
- VMware
tags:
- ESX4
- ESXi4
- NUMA
- VMware
- vSphere
- vSphere client
author: juan_manuel_rey
comments: true
---

The first time I installed an ESX 4 Update 1 on VMware Workstation an awful red message reporting some [NUMA](http://lse.sourceforge.net/numa/faq/) errors appeared on the main console screen.

[![](/images/esx4-plus_numa_error.png "ESX4 NUMA error")]({{site.url}}/images/esx4-plus_numa_error.png)

At that time I decided to ignore it. It didn't interfere with the normal functioning of the ESXs and since I never got again to the console of the ESX, I just fired up the VM in Workstation and then started to work from the vSphere Client, for a long time the error fall into the oblivion.

This week I decided to install a new ESX4 and couple of ESXi4 VMs in my home lab and the error appeared again and this time the geek inside me couldn't resist and after doing some research I found this [VMware Knowledge Base article](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1016141) which also pointed to a Dell document, both of them said that the error could be ignored because there is no real loss of service, something that I already knew. I finally found the solution in a [VMTN post](http://communities.vmware.com/thread/244537).

From the vSphere Client go to **Configuration -> Software -> Advanced Settings** and in the VMkernel area disable the **VMkernel.Boot.userNUMAInfo** setting.

[![](/images/numa.jpg "NUMA")]({{site.url}}/images/numa.jpg)

After that reboot your ESX and will see that the error has disappear.

I also noticed that the error is present on the virtualized ESXi but to see it from the ESXi console press Alt-F11 and you will get to a screen almost identical as the one from the first screenshot.

Juanma.
