---
title: HP Lefthand VSA minimum memory requirements
date: 2010-09-17
type: post
classes: wide
published: true
status: publish
categories:
- Storage
- Virtualization
tags:
- HP Lefthand
- P4000 VSA
- Storage
- VMware
author: juan_manuel_rey
comments: true
---

These week I've trying to stretch the virtualization resources of my homelab as much as possible. In my obsession to run as many VMs as possible I decided to lower the memory of some of them, including my storage appliances.

My VSAs are configured with various amounts of RAM ranging from 384MB to 1GB. I took the one I have in my laptop for demo purposes, powered it off, set the RAM to 256MB and fired it up again.

The VSA seemed to start without any problems and from the console everything looked fine.

[![](/assets/images/vsa_console1.jpg "VSA console")]({{site.url}}/assets/images/vsa_console1.jpg)

I started the CMC and quickly noticed that something was wrong, the status of the storage server was offline.

[![](/assets/images/storage_server_offline.jpg "storage_server_offline")]({{site.url}}/assets/images/storage_server_offline.jpg)

I then looked into the alerts area and found one saying that there was not enough ram to start the configured features.

[![](/assets/images/no-memory.jpg "Not enough memory")]({{site.url}}/assets/images/no-memory.jpg)

OK then, the VSA doesn't work with 256MB of RAM; so which value is the minimum required in order to run the storage services?

After looking into several docs I found the answer in the [P4000 Quick Start VSA user guide](http://bizsupport2.austin.hp.com/bc/docs/support/SupportManual/c02063198/c02063198.pdf). The minimum amount of RAM required is 384MB for the laptop version and 1GB for the ESX version. Also in the VSA Install and Configure Guide, that comes with the VSA, the following values are provided for the ESX version and for the new Hyper-V version:

- 500GB to 4.5TB - 1GB of RAM
- 4.5TB to 9TB - 2GB of RAM
- 9TB to 10TB - 3GB of RAM

After that I configured again the VSA with 384MB and the problem was fixed and the alarm disappeared.

Juanma.
