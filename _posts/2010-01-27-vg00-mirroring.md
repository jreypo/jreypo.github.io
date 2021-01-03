---
title: vg00 mirroring
date: 2010-01-27
type: post
classes: wide
published: true
status: publish
categories:
- HP-UX
- Sysadmin
tags:
- 11iv2
- 11iv3
- HP-UX
- Itanium
- LVM
- Mirror-UX
- PA-RISC
- Raid
- sysadmin
- systems administration
author: juan_manuel_rey
comments: true
---

This is a small cookbook about mirroring the `vg00` I've compiled throughout the years, well it's really more like a list with the commands but I believe it can be of usefulness for some of the newbies out there. It covers HP-UX 11.23 for PA-RISC and 11.23 and 11.31 for IA64.

### PA-RISC 11.23

First initialize the disk.

```
root@ayane:/# pvcreate -f -B /dev/rdsk/c0t6d0
```

Now make the disk bootable writing the LIF header.

```
root@ayane:/# mkboot -l /dev/dsk/c0t6d0
```

And the LIF files, I'm using the unenforced quorum option because in my example `vg00` has only two PVs.

```
root@ayane:/# mkboot -a 'hpux -lq' /dev/dsk/c0t6d0
```

Add the new PV to `vg00`.

```
root@ayane:/# vgextend vg00 /dev/dsk/c0t6d0
```

Create the mirrors of the logical volumes within `vg00` in the new PV.

```
root@ayane:/# for i in $(vgdisplay -v vg00 | grep "LV Name" | awk '{ print $3 };')
> do
> lvextend -m 1 $i /dev/dsk/c0t6d0
> done
```

When the mirror is finished.

```
root@ayane:/# lvlnboot -r /dev/vg00/lvol3 /dev/vg00
root@ayane:/# lvlnboot -b /dev/vg00/lvol1 /dev/vg00
root@ayane:/# lvlnboot -s /dev/vg00/lvol2 /dev/vg00
root@ayane:/# lvlnboot -d /dev/vg00/lvol2 /dev/vg00
```

Specify the new disk as alternate boot path and add it to `/stand/bootconf`.

```
root@ayane:/# setboot -a 0/0/0/3/0.6.0
root@ayane:/# cat /stand/bootconf
l  /dev/dsk/c0t5d0
l  /dev/dsk/c0t6d0
root@ayane:/#
```

And it's done. To check that everything correct.

```
root@ayane:/# vgdisplay -v vg00
--- Volume groups ---
VG Name                     /dev/vg00
VG Write Access             read/write
VG Status                   available
Max LV                      255
Cur LV                      10
Open LV                     10
Max PV                      16
Cur PV                      2
Act PV                      2
Max PE per PV               4384
VGDA                        4
PE Size (Mbytes)            32
Total PE                    8748
Alloc PE                    4076
Free PE                     4672
Total PVG                   0
Total Spare PVs             0
Total Spare PVs in use      0

 --- Logical volumes ---
.
.
.
 --- Physical volumes ---
 PV Name                     /dev/dsk/c0t5d0
 PV Status                   available
 Total PE                    4374
 Free PE                     2336
 Autoswitch                  On
 Proactive Polling           On

 PV Name                     /dev/dsk/c0t6d0
 PV Status                   available
 Total PE                    4374
 Free PE                     2336
 Autoswitch                  On
 Proactive Polling           On

root@ayane:/#
root@ayane:/# lvlnboot -v
Boot Definitions for Volume Group /dev/vg00:
Physical Volumes belonging in Root Volume Group:
 /dev/dsk/c0t5d0 (0/0/0/3/0.5.0) -- Boot Disk
 /dev/dsk/c0t6d0 (0/0/0/3/0.6.0) -- Boot Disk
Boot: lvol1     on:     /dev/dsk/c0t5d0
                        /dev/dsk/c0t6d0
Root: lvol3     on:     /dev/dsk/c0t5d0
                        /dev/dsk/c0t6d0
Swap: lvol2     on:     /dev/dsk/c0t5d0
                        /dev/dsk/c0t6d0
Dump: lvol2     on:     /dev/dsk/c0t5d0, 0

root@ayane:/#
root@ayane:/# setboot
Primary bootpath : 0/0/0/3/0.6.0
HA Alternate bootpath : 0/0/0/0/0.0.0
Alternate bootpath : 0/0/0/3/0.5.0

Autoboot is ON (enabled)
Autosearch is ON (enabled)

root@ayane:/#
root@ayane:/# lifls -l /dev/dsk/c0t6d0
volume ISL10 data size 7984 directory size 8 05/09/22 09:37:09
filename   type   start   size     implement  created
===============================================================
HPUX       -12928 584     1024     0          06/10/27 14:23:07
ISL        -12800 1608    242      0          05/09/22 09:37:09
AUTO       -12289 1856    1        0          05/09/22 09:37:09
PAD        -12290 1864    1468     0          05/09/22 09:37:09
LABEL      BIN    3336    8        0          07/07/17 19:42:29
root@ayane:/#
```

### Itanium 11.23

The procedure of mirroring `vg00` in an Itanium HP-UX 11.23, although shares some part with the PA-RISC differs in some critical area. The main difference is the partitioning of the disk, the boot disks in an Integrity server must have an specific layout with three partitions:

1.  EFI
2.  HPUX
3.  HPSP (HP Service Partition)

Preparation of the disk:

```
root@asoka:/# touch /tmp/partitionfile
root@asoka:/# cat <<EOF >> /tmp/partitionfile
> 3
> EFI 500MB
> HPUX 100%
> HPSP 400MB
> EOF
root@asoka:/# cat partitionfile
3
EFI 500MB
HPUX 100%
HPSP 400MB
root@asoka:/#
root@asoka:/# idisk -wqf /tmp/partitionfile /dev/rdsk/c1t1d0
```

Make it bootable and copy the AUTO file. As it can be viewed in the example below as `<number_of_partition>` has been added to the device in order to identify the partition in which the operation will be executed.

```
root@asoka:/# insf -eCdisk
root@asoka:/# mkboot -e -l /dev/rdsk/c1t1d0
root@asoka:/# echo "boot vmunix -lq" >> /tmp/AUTO.lq
root@asoka:/# efi_cp -d /dev/rdsk/c1t1d0s1 /tmp/AUTO.lq /EFI/HPUX/AUTO
```

Create the HPSP partition.

```
root@asoka:/# dd if=/dev/rdsk/c0t1d0s3 of=/dev/rdsk/c1t1d0s3 bs=1024k
```

Like in PA-RISC initialize the PV and add it to the VG.

```
root@asoka:/# pvreate -f -B /dev/rdsk/c1t1d0s2
root@asoka:/# vgextend vg00 /dev/dsk/c1t1d0s2
```

Mirror the Logical Volumes.

```
root@asoka:/# for i in $(vgdisplay -v vg00 | grep "LV Name" | awk '{ print $3 };')
> do
> lvextend -m 1 $i /dev/dsk/c1t1d0s2
> done
```

Set the content of the LABEL file, edit the `/stand/bootconf` like in the PA-RISC procedure and add the new disk as alternate HA boot path.

```
root@asoka:/# setboot -h <HW_PATH>
```

To check that everything is properly configured you can use the same commands as in PA-RISC and the command `idisk` to check the correct partitioning of the disk.

```
root@asoka:/# idisk -p /dev/rdsk/c1t1d0
idisk version: 1.32

EFI Primary Header:
        Signature                 = EFI PART
        Revision                  = 0x10000
        HeaderSize                = 0x5c
        HeaderCRC32               = 0xa498de56
        MyLbaLo                   = 0x1
        MyLbaHi                   = 0x0
        AlternateLbaLo            = 0x88aacbf
        AlternateLbaHi            = 0x0
        FirstUsableLbaLo          = 0x22
        FirstUsableLbaHi          = 0x0
        LastUsableLbaLo           = 0x88aac9c
        LastUsableLbaHi           = 0x0
        Disk GUID                 = 24e8312a-20cf-11dd-8001-d6217b60e588
        PartitionEntryLbaLo       = 0x2
        PartitionEntryLbaHi       = 0x0
        NumberOfPartitionEntries  = 0xc
        SizeOfPartitionEntry      = 0x80
        PartitionEntryArrayCRC32  = 0xae99dcc3

  Primary Partition Table (in 512 byte blocks):
    Partition 1 (EFI):
        Partition Type GUID       = c12a7328-f81f-11d2-ba4b-00a0c93ec93b
        Unique Partition GUID     = 24e83378-20cf-11dd-8002-d6217b60e588
        Starting Lba Lo            = 0x22
        Starting Lba Hi            = 0x0
        Ending Lba Lo              = 0xfa021
        Ending Lba Hi              = 0x0
    Partition 2 (HP-UX):
        Partition Type GUID       = 75894c1e-3aeb-11d3-b7c1-7b03a0000000
        Unique Partition GUID     = 24e83396-20cf-11dd-8003-d6217b60e588
        Starting Lba Lo            = 0xfa022
        Starting Lba Hi            = 0x0
        Ending Lba Lo              = 0x87e2821
        Ending Lba Hi              = 0x0
    Partition 3 (HPSP):
        Partition Type GUID       = e2a1e728-32e3-11d6-a682-7b03a0000000
        Unique Partition GUID     = 24e833b4-20cf-11dd-8004-d6217b60e588
        Starting Lba Lo            = 0x87e2822
        Starting Lba Hi            = 0x0
        Ending Lba Lo              = 0x88aa821
        Ending Lba Hi              = 0x0
root@asoka:/#
```

### Itanium 11.31

The 11.31 section will be short since the procedure is almost equal to the 11.23 one. You have to take into account that if you have migrated to the new agile view (and you should have ;-D ) a few things will change. The new agile view use `_p1`, `_p2` and `_p3` to identify the partitions of the disk instead of the classic `s1`, `s2` &  `s3 `` and the paths will change from  `dsk/rdsk `` to `disk/rdisk`, the names of the devices as well.

A few of examples will show it.

```
root@piroko:/# efi_cp -d /dev/rdisk/disk4_p1 /tmp/AUTO.lq /EFI/HPUX/AUTO

root@piroko:/# pvcreate -fB /dev/rdisk/disk4_p2
root@piroko:/# vgextend rootvg /dev/disk/disk_p2
root@piroko:/# for i in $(vgdisplay -v vg00| grep "LV Name" | awk '{ print $3 };')
> do
> lvextend -m 1 $i /dev/disk/disk4_p2
> done
```

And we are done. As always every comment or correction will be welcome.

See you next time.

Juanma.
