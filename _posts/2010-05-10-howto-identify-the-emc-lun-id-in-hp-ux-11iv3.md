---
title: Howto identify the EMC LUN ID in HP-UX 11iv3 - Symmetrix arrays
date: 2010-05-10
type: post
classes: wide
published: true
status: publish
categories:
- EMC
- HP-UX
- Storage
- Sysadmin
tags:
- 11iv2
- 11iv3
- DMX-3
- EMC
- EMC Powerpath
- HP-UX
- scsimgr
- Storage
- Symmetrix
author: juan_manuel_rey
comments: true
---

**DISCLAIMER NOTE:** This method is based only on my personal experience working with HP-UX 11iv2, 11iv3 and EMC Symmetrix. I tested it with near a hundred LUNs from a DMX-3 and with six different servers. As far as I know this isn't an official or supported procedure neither from EMC nor from HP.

Every time the storage people add a new LUN to your servers from an EMC disk array they provide you with a Logical device ID (or LUN ID) to identify the disk with PowerPath. If you are in HP-UX 11iv2 no problem here, just run a simple `powermt` command and look for the new LUN.

```
[root@totoro] / # powermt display dev=all | more
...
...
Symmetrix ID=000281150123
Logical device ID=0CED
state=alive; policy=SymmOpt; priority=0; queued-IOs=0
==============================================================================
---------------- Host ---------------   - Stor -   -- I/O Path -  -- Stats ---
###  HW Path                I/O Paths    Interf.   Mode    State  Q-IOs Errors
==============================================================================
20 0/0/10/1/0.11.15.0.0.1.3 c7t1d3 SP A0 active alive 0 1
23 0/0/10/1/0.11.47.0.0.1.3 c8t1d3 SP B0 active alive 0 1
26 1/0/8/1/0.21.15.0.0.1.3 c10t1d3 SP A1 active alive 0 1
29 1/0/8/1/0.21.47.0.0.1.3 c11t1d3 SP B1 active alive 0 1
...
...
```

But if you are in 11.31 you will find a small problem to perform this. PowerPath is not recommended in HP-UX 11iv3 because it can cause conflicts with the new native multipathing of v3.

You can use the trick of doing a simple `ls -ltr` in the `/dev/disk` directory just after the hardware scan and the device file creation, but this way is valid only if you have one or two disks with the same size. What if you have several disks with multiple sizes and want to use each disk for a different VG and/or task? The storage people will only provide the LUN IDs but you will not have the tool to match those IDs with your disks.

Fortunately there is way to circumvent the lack of PowerPath in 11iv3. We are going to use the same disk as in the previous example, the 0CED.

First get the disks serial number with `scsimgr`.

```
[root@totoro] / # scsimgr get_attr -D /dev/rdisk/disk30 -a serial_number

 SCSI ATTRIBUTES FOR LUN : /dev/rdisk/disk30

name = serial_number
current = "100123CED000"
default =
saved =
```

Take note of the serial number `100123CED000`.

As you can see the last the last three digits of the LUN ID (`CED`) are included in the disk serial number and if look carefully will see also the four last digits the Symmetrix ID (`0123`) just after the LUN ID.

Juanma.
