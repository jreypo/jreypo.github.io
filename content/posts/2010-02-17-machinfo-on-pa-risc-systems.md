---
title: Machinfo on PA-RISC systems
date: 2010-02-17
categories:
- HP-UX
- PA-RISC
tags:
- 11iv1
- 11iv2
- commands
- HP-UX
- machinfo
- PA-RISC
showComments: true
---

When the HP-UX Itanium version was released  it came with a very handy tool named [`machinfo`](http://docs.hp.com/en/B2355-60130/machinfo.1.html "Machinfo MAN page"). This command prints number and type of CPUs, amount of memory, firmware revision, serial number, machine id and many more useful information about the server. Sadly on PA-RISC we didn't have something similar, of course there is [`print_manifest`](http://www.docs.hp.com/en/5992-6587/5992-6587.pdf "Ignite-UX Reference") and I have to say that it's quite a tool but I always loved the simplicity of `machinfo`.

Dozens of posts have been thrown throughout the years on [ITRC Forums](http://forums11.itrc.hp.com/service/forums/home.do "ITRC Forums") asking for a PA-RISC `machinfo` version and many workarounds and even some binaries have been posted.

I tried a lot of them but since I discovered [Patrick Reut](http://forums11.itrc.hp.com/service/forums/publicProfile.do?userId=CA687674&forumId=1)'s version a couple of years ago I never used another one. It works on 11.11 and 11.23 systems, I tried on 10.20 but didn't wok. The binary can found in this [post](http://forums11.itrc.hp.com/service/forums/questionanswer.do?admit=109447626+1266396818246+28353475&threadId=1104988) as an attachment.

Here it an example output of the command:

```text
root@tst1:/# which machinfo
/usr/contrib/bin/machinfo
root@tst1:/# ll /usr/contrib/bin/machinfo
lrwx------   1 root       sys             34 Feb 17 10:15 /usr/contrib/bin/machinfo@ -> /usr/local/scripts/parisc_machinfo
root@tst1:/#
root@tst1:/# machinfo
OS info:
 sysname  = HP-UX
 nodename = sigtst1
 release  = B.11.23
 version  = U
 machine  = 9000/800
 idnumber = 11111111111

Platform info:
 model string = "9000/800/rp7420"
 machine id number = XXXXXXXXXXXXXX
 machine serial number = DEHxxxxxxxx

CPU info
 Number of CPUs = 4
 Clock speed    = 1000 MHz
 processor family: 532 pa-2.0
 processor model:  20 PA8900

Firmware info:
 Firmware revision = 22.2

Memory = 8165 MB (7 GB)
Disk info:
c0t6d0 : HP 146 GMA  (size = 146GB)
c0t5d0 : HP 146 GMA  (size = 146GB)
c45t0d1 : HP      HS  (size = 118GB)
c45t0d4 : HP      HS  (size = 118GB)
c45t0d5 : HP      HS  (size = 118GB)
c45t0d7 : HP      HS  (size = 15GB)
c45t1d0 : HP      HS  (size = 15GB)
c47t0d1 : HP      HS  (size = 118GB)
c47t0d4 : HP      HS  (size = 118GB)
c47t0d5 : HP      HS  (size = 118GB)
c47t0d7 : HP      HS  (size = 15GB)
c47t1d0 : HP      HS  (size = 15GB)

root@tst1:/#
```

Juanma.
