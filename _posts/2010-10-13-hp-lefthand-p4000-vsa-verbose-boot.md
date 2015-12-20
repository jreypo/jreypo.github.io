---
layout: post
title: HP Lefthand P4000 VSA verbose boot
date: 2010-10-13 21:06:16.000000000 +02:00
type: post
published: true
status: publish
categories:
- Storage
- Virtualization
tags:
- HP Lefthand
- P4000 VSA
author: juan_manuel_rey
---

If you are a user of the P4000 VSA you'll be use to the quiet boot sequence of the SAN/iQ software. Just a couple of messages until you get the login prompt.

[![](/images/p4000-vsa_clean_boot.jpg "P4000-VSA_clean_boot")]({{site.url}}/images/p4000-vsa_clean_boot.jpg)

But how about if anyone want to watch the whole boot process to check error messages or something alike? There is an easy and simple solution, at the beginning of the boot sequence press *ESC* in order to stop the bootloader and when the `boot:` prompt appears type `vga` and press Enter.

After that you will have a normal boot process like with any other Linux system.

[![](/images/p4000-vsa_full_boot.png "P4000-VSA_full_boot")]({{site.url}}/images/p4000-vsa_full_boot.png)

Juanma.
