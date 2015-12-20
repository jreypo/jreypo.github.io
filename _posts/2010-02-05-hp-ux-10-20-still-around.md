---
layout: post
title: HP-UX 10.20, still around :-)
date: 2010-02-05 14:02:26.000000000 +01:00
type: post
published: true
status: publish
categories:
- HP-UX
- PA-RISC
tags:
- '10.20'
- D330
- HP-UX
- HP9000
- PA-RISC
author: juan_manuel_rey
---

Ten years or so ago the first HP-UX I worked with was a 10.20, it was an old D230 HP9000 server.

I never thought I would find a still running 10.20 server but when I came to the customer I am currently assigned I had a nice surprise. Hidden in the lab room next to the data-center was an old HP9000 D330 server with HP-UX 10.20 installed on it. And you know what? It's still in production running an ancient version of Informix for a legacy service.

The server has one PA-7200LC 160MHz CPU, 192MB of RAM and four Seagate ST34572WC Ultra SCSI 4GB disks. Since I discovered it the Informix has become my favorite server above the Superdomes or the Integrity Blades and I even found `bash` and `lsof` depots for 10.20 :-D

Bash can be found on this [FTP site](http://ftp.hi.is/pub/unix/hpux/) and `lsof` and a lot more OpenSource software for 10.20 and 11.00 can be found at [Merjin's HP-UX](http://mirrors.develooper.com/hpux/downloads.html).

Here it is screenshot of a PuTTY session and a couple of pictures of the beast taken with my E71:

[![HP-UX 10.20](/images/10_20.jpg "HP-UX 10.20")]({{site.url}}/images/10_20.jpg)

[![D330](/images/d330_view.jpg "D330")]({{site.url}}/images/d330_view.jpg)

[![D330 Disks](/images/d330_disks.jpg "D330_disks")]({{site.url}}/images/d330_disks.jpg)

Does anyone of you have an old HP-UX 10.20 still around?

Juanma.
