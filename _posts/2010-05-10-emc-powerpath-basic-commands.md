---
layout: post
title: EMC PowerPath basic commands
date: 2010-05-10
type: post
published: true
status: publish
categories:
- EMC
- HP-UX
- Storage
- Sysadmin
tags:
- EMC
- EMC Powerpath
- HP-UX
- powermt
- Storage
- sysadmin
- systems administration
author: juan_manuel_rey
comments: true
---

PowerPath is a multipathing software for Unix operating systems from EMC. If you have ever worked or you are going to work in an environment that includes EMC storage systems it is more than sure that PowerPath will be installed in the Unix hosts.

Following are some notes and tips I've been creating since the very first time I found Powerpath, of course this isn't a full user guide but a sort of personal quick reference. I decide to put it here in the hope that it will be helpful to anyone and for my personal use.

-   Show `powermt` command version

```
[root@totoro] / # powermt version
EMC powermt for PowerPath (c) Version 5.1.0 (build 160)
```

-   Display PowerPath configuration.

```
[root@totoro] / # powermt display
Symmetrix logical device count=898
CLARiiON logical device count=0
Hitachi logical device count=0
Invista logical device count=0
HP xp logical device count=0
Ess logical device count=0
HP HSx logical device count=0
==============================================================================
----- Host Bus Adapters ---------  ------ I/O Paths -----  ------ Stats ------
###  HW Path                       Summary   Total   Dead  IO/Sec Q-IOs Errors
==============================================================================
 5 0/2/1/0.101.16.19.0           optimal      61      0       -     0      0
 6 0/2/1/0.101.16.19.1           optimal     102      0       -     0      0
 7 0/2/1/0.101.16.19.2           optimal      97      0       -     0      0
 8 0/2/1/0.101.16.19.3           optimal     113      0       -     0      0
 9 0/2/1/0.101.16.19.4           optimal      82      0       -     0      0
 11 0/2/1/0.101.43.19.0           optimal     128      0       -     0      0
 12 0/2/1/0.101.43.19.1           optimal      49      0       -     0      0
 13 0/2/1/0.101.43.19.2           optimal      57      0       -     0      0
 14 0/2/1/0.101.43.19.3           optimal      83      0       -     0      0
 15 0/2/1/0.101.43.19.4           optimal      74      0       -     0      0
 16 0/2/1/0.101.43.19.5           optimal      33      0       -     0      0
 17 0/2/1/0.101.43.19.6           optimal      19      0       -     0      0
 19 0/5/1/0.102.16.19.0           optimal      61      0       -     0      0
 20 0/5/1/0.102.16.19.1           optimal     102      0       -     0      0
 21 0/5/1/0.102.16.19.2           optimal      97      0       -     0      0
 22 0/5/1/0.102.16.19.3           optimal     113      0       -     0      0
 23 0/5/1/0.102.16.19.4           optimal      82      0       -     0      0
 25 0/5/1/0.102.43.19.0           optimal     128      0       -     0      0
 26 0/5/1/0.102.43.19.1           optimal      49      0       -     0      0
 27 0/5/1/0.102.43.19.2           optimal      57      0       -     0      0
 28 0/5/1/0.102.43.19.3           optimal      83      0       -     0      0
 29 0/5/1/0.102.43.19.4           optimal      74      0       -     0      0
 30 0/5/1/0.102.43.19.5           optimal      33      0       -     0      0
 31 0/5/1/0.102.43.19.6           optimal      19      0       -     0      0

[root@totoro] / #
```

-   Check for death paths and remove them.

```
[root@sheldon] / # powermt display
Symmetrix logical device count=34
CLARiiON logical device count=0
Hitachi logical device count=0
Invista logical device count=0
HP xp logical device count=0
Ess logical device count=0
HP HSx logical device count=0
==============================================================================
----- Host Bus Adapters ---------  ------ I/O Paths -----  ------ Stats ------
###  HW Path                       Summary   Total   Dead  IO/Sec Q-IOs Errors
==============================================================================
 17 UNKNOWN                       failed        1      1       -     0      0
 31 UNKNOWN                       failed        1      1       -     0      0
 37 1/0/14/1/0.109.85.19.0        optimal      32      0       -     0      0
 39 0/0/14/1/0.110.85.19.0        optimal      32      0       -     0      0

[root@sheldon] / # powermt check
Warning: Symmetrix device path c17t9d6 is currently dead.
Do you want to remove it (y/n/a/q)? y
Warning: Symmetrix device path c31t9d6 is currently dead.
Do you want to remove it (y/n/a/q)? y
[root@sheldon] / #
```

-   List all devices.

```
[root@totoro] / # powermt display dev=all
```
-   Remove all devices.

```
[root@totoro] / # powermt remove dev=all
```

-   Add a new disk in HP-UX, configure it and save the config:

After a rescan of the disks with `ioscan` and the creation of the device files with `insf` run the following command  to add the new disk to PowerPath

```
[root@totoro] / # powermt config
```

Now display all the devices and look the for the `Logical device ID` of the disk.

```
[root@totoro] / # powermt display dev=all | more
...
...
Symmetrix ID=000287750035
Logical device ID=0004
state=alive; policy=SymmOpt; priority=0; queued-IOs=0
==============================================================================
---------------- Host ---------------   - Stor -   -- I/O Path -  -- Stats ---
###  HW Path                I/O Paths    Interf.   Mode    State  Q-IOs Errors
==============================================================================
20 0/0/10/1/0.11.15.0.0.1.3 c20t1d3 SP A0 active alive 0 1
23 0/0/10/1/0.11.47.0.0.1.3 c23t1d3 SP B0 active alive 0 1
26 1/0/8/1/0.21.15.0.0.1.3 c26t1d3 SP A1 active alive 0 1
29 1/0/8/1/0.21.47.0.0.1.3 c29t1d3 SP B1 active alive 0 1
...
...
```

If everything went fine save the config.

```
[root@totoro] / # powermt save
```

And these are the most common tasks I've been doing with PowerPath. I'll try to put some order into my notes and personal how-to files and write more posts like this one.

Juanma.
