---
layout: post
title: Mirroring VxVM root disk group
date: 2010-08-16 09:46:47.000000000 +02:00
type: post
published: true
status: publish
categories:
- HP-UX
- Sysadmin
tags:
- EFI
- HP-UX
- sysadmin
- systems administration
- vxrootmir
- VxVM
author: juan_manuel_rey
comments: true
---

Mirroring the root disk group of an HP-UX server is a very straightforward process that can be performed by simply use the `vxrootmir` command an use the new mirror's disk as the argument. This handy utility will create and configure the boot and the VxVM structures and will mirror the volumes.

An example will explain better the whole process. Launch the mirroring operation.

```
root@robin:~# /etc/vx/bin/vxrootmir -v -b disk22
VxVM vxrootmir INFO V-5-2-2501 11:38: Gathering information on the current VxVM root configuration
VxVM vxrootmir INFO V-5-2-2441 11:38: Checking specified disk(s) for usability
VxVM vxrootmir INFO V-5-2-2566 11:38: Preparing disk disk22_p2 as a VxVM root disk
VxVM vxrootmir INFO V-5-2-3766 11:38: Disk disk22_p2 is now EFI partitioned disk disk22_p2
VxVM vxrootmir INFO V-5-2-2410 11:38: Adding disk disk22_p2 to rootdg as DM rootdisk02
VxVM vxrootmir INFO V-5-2-1646 11:38: Mirroring all volumes on root disk
VxVM vxrootmir INFO V-5-2-1648 11:38: Mirroring volume standvol
VxVM vxrootmir INFO V-5-2-1648 11:39: Mirroring volume swapvol
VxVM vxrootmir INFO V-5-2-1648 11:40: Mirroring volume rootvol
VxVM vxrootmir INFO V-5-2-1648 11:40: Mirroring volume homevol
VxVM vxrootmir INFO V-5-2-1648 11:40: Mirroring volume optvol
VxVM vxrootmir INFO V-5-2-1648 11:43: Mirroring volume tmpvol
VxVM vxrootmir INFO V-5-2-1648 11:43: Mirroring volume usrvol
VxVM vxrootmir INFO V-5-2-1648 11:44: Mirroring volume varvol
VxVM vxrootmir INFO V-5-2-1648 11:45: Mirroring volume crashvol
VxVM vxrootmir INFO V-5-2-2462 11:46: Current setboot values:
VxVM vxrootmir INFO V-5-2-2569 11:46: Primary:  0/0/0/0.0x8.0x0 /dev/rdisk/disk20
VxVM vxrootmir INFO V-5-2-2416 11:46: Alternate:        0/0/0/0.0x0.0x0 /dev/rdisk/disk4
VxVM vxrootmir INFO V-5-2-2551 11:46: Making mirror disk disk22 (/dev/rdisk/disk22) the alternate boot disk
Alternate boot path set to 0/0/0/0.0x9.0x0 (/dev/rdisk/disk22)
VxVM vxrootmir INFO V-5-2-1616 11:46: Disk disk22 is now a mirrored root disk
root@robin:~#
```

After the mirror operation is finished verify the contents of the `rootdg`.

```
root@robin:~# vxprint -htg rootdg
DG NAME         NCONFIG      NLOG     MINORS   GROUP-ID
ST NAME         STATE        DM_CNT   SPARE_CNT         APPVOL_CNT
DM NAME         DEVICE       TYPE     PRIVLEN  PUBLEN   STATE
RV NAME         RLINK_CNT    KSTATE   STATE    PRIMARY  DATAVOLS  SRL
RL NAME         RVG          KSTATE   STATE    REM_HOST REM_DG    REM_RLNK
CO NAME         CACHEVOL     KSTATE   STATE
VT NAME         RVG          KSTATE   STATE    NVOLUME
V  NAME         RVG/VSET/CO  KSTATE   STATE    LENGTH   READPOL   PREFPLEX UTYPE
PL NAME         VOLUME       KSTATE   STATE    LENGTH   LAYOUT    NCOL/WID MODE
SD NAME         PLEX         DISK     DISKOFFS LENGTH   [COL/]OFF DEVICE   MODE
SV NAME         PLEX         VOLNAME  NVOLLAYR LENGTH   [COL/]OFF AM/NM    MODE
SC NAME         PLEX         CACHE    DISKOFFS LENGTH   [COL/]OFF DEVICE   MODE
DC NAME         PARENTVOL    LOGVOL
SP NAME         SNAPVOL      DCO
EX NAME         ASSOC        VC                       PERMS    MODE     STATE
SR NAME         KSTATE

dg rootdg       default      default  4466000  1276076559.38.robin

dm rootdisk01   disk20_p2    auto     1024     40035232 -
dm rootdisk02   disk22_p2    auto     1024     40035232 -

v  crashvol     -            ENABLED  ACTIVE   4194304  SELECT    -        fsgen
pl crashvol-01  crashvol     ENABLED  ACTIVE   4194304  CONCAT    -        RW
sd rootdisk01-09 crashvol-01 rootdisk01 28778496 4194304 0        c0t8d0s2 ENA
pl crashvol-02  crashvol     ENABLED  ACTIVE   4194304  CONCAT    -        RW
sd rootdisk02-09 crashvol-02 rootdisk02 28778496 4194304 0        c0t9d0s2 ENA

v  homevol      -            ENABLED  ACTIVE   155648   SELECT    -        fsgen
pl homevol-01   homevol      ENABLED  ACTIVE   155648   CONCAT    -        RW
sd rootdisk01-04 homevol-01  rootdisk01 7077888 155648  0         c0t8d0s2 ENA
pl homevol-02   homevol      ENABLED  ACTIVE   155648   CONCAT    -        RW
sd rootdisk02-04 homevol-02  rootdisk02 7077888 155648  0         c0t9d0s2 ENA

v  optvol       -            ENABLED  ACTIVE   9560064  SELECT    -        fsgen
pl optvol-01    optvol       ENABLED  ACTIVE   9560064  CONCAT    -        RW
sd rootdisk01-05 optvol-01   rootdisk01 7233536 9560064 0         c0t8d0s2 ENA
pl optvol-02    optvol       ENABLED  ACTIVE   9560064  CONCAT    -        RW
sd rootdisk02-05 optvol-02   rootdisk02 7233536 9560064 0         c0t9d0s2 ENA

v  rootvol      -            ENABLED  ACTIVE   1048576  SELECT    -        root
pl rootvol-01   rootvol      ENABLED  ACTIVE   1048576  CONCAT    -        RW
sd rootdisk01-03 rootvol-01  rootdisk01 6029312 1048576 0         c0t8d0s2 ENA
pl rootvol-02   rootvol      ENABLED  ACTIVE   1048576  CONCAT    -        RW
sd rootdisk02-03 rootvol-02  rootdisk02 6029312 1048576 0         c0t9d0s2 ENA

v  standvol     -            ENABLED  ACTIVE   1835008  SELECT    -        fsgen
pl standvol-01  standvol     ENABLED  ACTIVE   1835008  CONCAT    -        RW
sd rootdisk01-01 standvol-01 rootdisk01 0      1835008  0         c0t8d0s2 ENA
pl standvol-02  standvol     ENABLED  ACTIVE   1835008  CONCAT    -        RW
sd rootdisk02-01 standvol-02 rootdisk02 0      1835008  0         c0t9d0s2 ENA

v  swapvol      -            ENABLED  ACTIVE   4194304  SELECT    -        swap
pl swapvol-01   swapvol      ENABLED  ACTIVE   4194304  CONCAT    -        RW
sd rootdisk01-02 swapvol-01  rootdisk01 1835008 4194304 0         c0t8d0s2 ENA
pl swapvol-02   swapvol      ENABLED  ACTIVE   4194304  CONCAT    -        RW
sd rootdisk02-02 swapvol-02  rootdisk02 1835008 4194304 0         c0t9d0s2 ENA

v  tmpvol       -            ENABLED  ACTIVE   524288   SELECT    -        fsgen
pl tmpvol-01    tmpvol       ENABLED  ACTIVE   524288   CONCAT    -        RW
sd rootdisk01-06 tmpvol-01   rootdisk01 16793600 524288 0         c0t8d0s2 ENA
pl tmpvol-02    tmpvol       ENABLED  ACTIVE   524288   CONCAT    -        RW
sd rootdisk02-06 tmpvol-02   rootdisk02 16793600 524288 0         c0t9d0s2 ENA

v  usrvol       -            ENABLED  ACTIVE   6217728  SELECT    -        fsgen
pl usrvol-01    usrvol       ENABLED  ACTIVE   6217728  CONCAT    -        RW
sd rootdisk01-07 usrvol-01   rootdisk01 17317888 6217728 0        c0t8d0s2 ENA
pl usrvol-02    usrvol       ENABLED  ACTIVE   6217728  CONCAT    -        RW
sd rootdisk02-07 usrvol-02   rootdisk02 17317888 6217728 0        c0t9d0s2 ENA

v  varvol       -            ENABLED  ACTIVE   5242880  SELECT    -        fsgen
pl varvol-01    varvol       ENABLED  ACTIVE   5242880  CONCAT    -        RW
sd rootdisk01-08 varvol-01   rootdisk01 23535616 5242880 0        c0t8d0s2 ENA
pl varvol-02    varvol       ENABLED  ACTIVE   5242880  CONCAT    -        RW
sd rootdisk02-08 varvol-02   rootdisk02 23535616 5242880 0        c0t9d0s2 ENA
root@robin:~#
```

Then check the contents of the EFI partition and the `LABEL` file of the mirror disk.

```
root@robin:~# efi_ls -d /dev/rdisk/disk22_p1 /efi/hpux
FileName                             Last Modified             Size
.                                      6/ 9/2010                  0
..                                     6/ 9/2010                  0
hpux.efi                               6/ 9/2010             698356
nbp.efi                                6/ 9/2010              31232
AUTO                                   6/ 9/2010                 12

total space 523218944 bytes, free space 520163328 bytes

root@robin:~# vxvmboot -v /dev/rdisk/disk22_p2

LIF Label File @ (1k) block # 834 on VxVM Disk /dev/rdsk/c0t9d0s2:
Label Entry: 0, Boot Volume start:     3168; length: 1792 MB
Label Entry: 1, Root Volume start:  6032480; length: 1024 MB
Label Entry: 2, Swap Volume start:  1838176; length: 4096 MB
Label Entry: 3, Dump Volume start:  1838176; length: 4096 MB
root@robin:~#
```

Verify that the new disk has been set as the alternate boot device.

```
root@robin:~# setboot -v
Primary bootpath : 0/0/0/0.0x8.0x0 (/dev/rdisk/disk20)
HA Alternate bootpath :
Alternate bootpath : 0/0/0/0.0x9.0x0 (/dev/rdisk/disk22)

Autoboot is ON (enabled)
setboot: error accessing firmware - Function is not available
The firmware of your system does not support querying or changing the SpeedyBoot
settings.
root@robin:~#
```

Finally reboot the server and from the EFI boot manager force the boot from the mirrored disk.

Juanma.
