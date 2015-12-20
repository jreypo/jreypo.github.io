---
layout: post
title: New Generation Integrity Servers
date: 2010-05-05 11:52:27.000000000 +02:00
type: post
published: true
status: publish
categories:
- HP
- Itanium
tags:
- BL860c_i2
- BL870c_i2
- BL890c_i2
- HP
- Integrity Servers
- Itanium
- rx2800
- Superdome 2
- Tukwila
author: juan_manuel_rey
---

Last week was, without any doubt, one of the most exciting of the year. The new Integrity Servers have been finally unveiled.

This new whole line of Integrity machines are based on **Tukwila**, the latest iteration of the Itanium processor line which was presented by Intel early this year, and with one exception all of them are based in the blade form factor. Let's take a quick look of the new servers.

### Entry-level

In this area, and as the only rack server of the new line, we have the rx28000, at first look it seems no more than a remake of the rx2660 but if you go deeper will find a powerful machine with 2 Quad-core or Dual-core Itanium 9300 processors and a maximum of 192GB of RAM.

That's a considerable amount of power for a server of this kind. I personally like this server and have to convince my manager to kindly donate one for my home lab ;-)

### Mid-range

In the mid-range line there are three beautiful babies named BL860c_i2, BL870c_i2 and BL890c_i2.

The key for this new servers is modularity, BL860c_i2 is the base of her bigger sisters. HP has developed a new piece of hardware known as Integrity Blade Link Assembly which makes possible to combine  blade modules. The 870 is composed by two blade modules and the 890 by four. The 860 is no more than a single blade module with a single Link Assembly on its front. This way of combining the blades makes the 890 the only 8 socket blade currently available.

[![](/images/blade_link_assembly.jpg "blade_link_assembly")]({{site.url}}/images/blade_link_assembly.jpg)

The 870 and the 890 with 16 and 32 cores respectively are the logical replacement for the rx7640 and rx8640 but as many people have been saying since they were publicly presented there is of  the OLAR question or really the apparently lack of OLAR which in fact was one of the key features of the mid-range cell-based Integrity servers. We'll see how this issue is solved.

### High-End

The new rx2800 and the new blades are great but the real shock for everybody came when HP announced the new Superdome 2. Ladies and gentlemen the new mission critical computing era is here, forget those fat and proprietary racks, forget everything you know about high-end servers and be welcome to the blade
land.

[![Superdome 2](/images/sd2.jpg "Superdome 2")]({{site.url}}/images/sd2.jpg)

This new version of the HP flagship is based on the blade concept. Instead of cells we have cell-blades inside a new 18U enclosure based in the HP C7000 Blade Enclosure. Just remember one word... **commonality**. The new Superdome 2 will share a lot of parts with the C7000 and can be also managed through the same tools like the Onboard Administrator.

The specs of this baby are astonishing and during the presentation at the HP Technology At Work event four different configurations were outlined ranging from 8 sockets/32 cores in four blade-cells to a maximum of 64 sockets/256 cores in 32 cell-blades distributed through four enclosures in two racks. Like I said, astonishing :-D

There have been a lot rumors during last year about HP-UX and Itanium future mainly because the delays of the Tukwilla processor. The discussion has recently reach [ITRC](http://forums11.itrc.hp.com/service/forums/questionanswer.do?admit=109447626+1272648125017+28353475&threadId=1416340).

But if any of you had doubts about HP-UX future I firmly believe that HP sent a clear message on the opposite direction. HP-UX is probably the more robust and reliable Unix in the enterprise arena. And to be serious, what are you going to use to replace it? Linux? Solaris? Please ;-)

Juanma.
