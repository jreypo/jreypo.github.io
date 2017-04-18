---
layout: post
title: Understanding RAID management in Linux
date: 2010-11-24
type: post
published: true
status: publish
categories:
- Linux
tags:
- device-mapper
- dmraid
- dmsetup
- EVMS
- Linux
- Linux LVM
- MD
- mdadm
- Raid
author: juan_manuel_rey
comments: true
---

The first thing you must learn about RAID technologies in Linux is that they have nothing in common with HP-UX, and I mean nothing! Yes there is LVM but that's all, the mirror of a volume group for example is not done through LVM commands, in fact you are not going to mirror a volume group but the block device/s where the volume group resides.

There are two tools to manage RAID in Linux.

-   `dmraid`
-   `mdadm`

`dmraid` is used to discover and activate software (ATA)RAID arrays, commonly known as fakeRAID, and `mdadm` is used to manage Linux Software RAID devices.

### dmraid

`dmraid`, uses `libdevmapper` and the device-mapper kernel driver to perform all the tasks.

The device-mapper is a component of the Linux Kernel. This the way the Linux Kernel do all the block device management. It maps a block device onto another and forms the base of volume management ([LVM2](http://sources.redhat.com/lvm2/ "LVM2") and [EVMS](http://evms.sourceforge.net/ "EVMS")) and software raid. Multipathing support is also provided through the device-mapper.
Device-mapper support is present in 2.6 kernels although there are patches for the most recent versions of 2.4 kernel version.

`dmraid` supports several array types.

```
[root@caladan ~]# dmraid -l
asr     : Adaptec HostRAID ASR (0,1,10)
ddf1    : SNIA DDF1 (0,1,4,5,linear)
hpt37x  : Highpoint HPT37X (S,0,1,10,01)
hpt45x  : Highpoint HPT45X (S,0,1,10)
isw     : Intel Software RAID (0,1)
jmicron : JMicron ATARAID (S,0,1)
lsi     : LSI Logic MegaRAID (0,1,10)
nvidia  : NVidia RAID (S,0,1,10,5)
pdc     : Promise FastTrack (S,0,1,10)
sil     : Silicon Image(tm) Medley(tm) (0,1,10)
via     : VIA Software RAID (S,0,1,10)
dos     : DOS partitions on SW RAIDs
[root@caladan ~]#
```

Following are a couple of examples to show `dmraid` operation.

#### Array discovering

```
[root@caladan ~]# dmraid -r
/dev/dm-46: hpt45x, "hpt45x_chidjhaiaa-0", striped, ok, 320172928 sectors, data@ 0
/dev/dm-50: hpt45x, "hpt45x_chidjhaiaa-0", striped, ok, 320172928 sectors, data@ 0
/dev/dm-54: hpt45x, "hpt45x_chidjhaiaa-1", striped, ok, 320172928 sectors, data@ 0
/dev/dm-58: hpt45x, "hpt45x_chidjhaiaa-1", striped, ok, 320172928 sectors, data@ 0

[root@caladan ~]#
```

#### Activate all discovered arrays

```
[root@caladan ~]# dmraid -ay
```

#### Deactivate all discovered arrays

```
[root@caladan ~]# dmraid -an
```

### mdadm

`mdadm`, is a tool to manage the Linux software RAID arrays. This tool has nothing to do with the device-mapper, in fact the device-mapper is not aware of the RAID arrays created with `mdadm`.

To illustrate this take a look at the screenshot below. I created a RAID1 device, `/dev/md0`, I then show its configuration with  `mdadm --detail`. Later with *dmsetup ls* I list all the block devices seen by the device-mapper, as you can see there is no reference to `/dev/md0`.

[![](/images/dmsetup-mdadm1.png "dmsetup mdadm")]({{site.url}}/images/dmsetup-mdadm1.png)

Instead `mdadm` uses the MD (Multiple Devices) device driver, this driver provides virtual devices created from another independent devices. Currently the MD driver supports the following RAID levels and configurations

-   RAID1
-   RAID4
-   RAID5
-   RAID6
-   RAID0
-   LINEAR (a concatenated array)
-   MULTIPATH
-   FAULTY (an special failed array type for testing purposes)

The configuration of the MD devices is contained in the `/etc/mdadm.conf` file.

```
[root@caladan ~]# cat mdadm.conf
ARRAY /dev/md1 level=raid5 num-devices=3 spares=1 UUID=5c9d6a69:4a0f120b:f6b02789:3bbc8698
ARRAY /dev/md0 level=raid1 num-devices=2 UUID=b36f1b1c:87cf9497:73b81e8c:79ee3c44
[root@caladan ~]#
```

The `mdadm` tool has seven operation modes.

1.  Assemble
2.  Build
3.  Create
4.  Manage
5.  Misc
6.  Follow or Monitor
7.  Grow

A more detailed description of every major operation mode is provided in `mdadm` man page.

Finally below are examples of some of the more common operations with `mdadm`.

#### Create a RAID1 array

```
[root@caladan ~]# mdadm --create /dev/md1 --verbose --level raid1 --raid-devices 2 /dev/sd[de]1
mdadm: size set to 1044096K
mdadm: array /dev/md1 started.
[root@caladan ~]#
```

#### Get detailed configuration of the array

```
[root@caladan ~]# mdadm --query --detail /dev/md1
/dev/md1:
            Version : 00.90.01
      Creation Time : Tue Nov 23 22:37:05 2010
         Raid Level : raid1
         Array Size : 1044096 (1019.80 MiB 1069.15 MB)
        Device Size : 1044096 (1019.80 MiB 1069.15 MB)
       Raid Devices : 2
      Total Devices : 2
    Preferred Minor : 1
        Persistence : Superblock is persistent

        Update Time : Tue Nov 23 22:37:11 2010
              State : clean
     Active Devices : 2
    Working Devices : 2
     Failed Devices : 0
      Spare Devices : 0

               UUID : c1893118:c1327582:7dc3a667:aa87dfeb
             Events : 0.2

        Number   Major   Minor   RaidDevice State
           0       8       49        0      active sync   /dev/sdd1
           1       8       65        1      active sync   /dev/sde1
[root@caladan ~]#
```

#### Destroy the array

```
[root@caladan ~]# mdadm --remove /dev/md1
[root@caladan ~]# mdadm --stop /dev/md1
[root@caladan ~]# mdadm --detail /dev/md1
mdadm: md device /dev/md1 does not appear to be active.
[root@caladan ~]#
```

#### Create a RAID5 array with an spare device

```
[root@caladan ~]# mdadm --create /dev/md1 --verbose --level raid5 --raid-devices 3 --spare-devices 1 /dev/sd[def]1 /dev/sdg1
mdadm: array /dev/md1 started
[root@caladan ~]#
```

#### Check for the status of a task into the /proc/mdstat file.

```
[root@caladan ~]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid6 sdi1[7] sdh1[6] sdg1[5] sdf1[4] sde1[3] sdd1[2] sdc1[1] sdb1[0]
             226467456 blocks level 6, 64k chunk, algorithm 2 [8/8] [UUUUUUUU]
             [=========>...........]  resync = 49.1% (18552320/37744576) finish=11.4min speed=27963K/sec

unused devices: <none>
[root@caladan ~]#
```

#### Generate the mdadm.conf file from the current active devices.

```
[root@caladan ~]# mdadm --detail --scan
ARRAY /dev/md1 level=raid5 num-devices=3 spares=1 UUID=5c9d6a69:4a0f120b:f6b02789:3bbc8698
ARRAY /dev/md0 level=raid1 num-devices=2 UUID=b36f1b1c:87cf9497:73b81e8c:79ee3c44
[root@caladan ~]# mdadm --detail --scan >> mdadm.conf
```

As a final thought, my recommendation is that if there is hardware RAID controller available, like the HP Smart Array P400 for example, go hard-RAID five by five and if not always use `mdadm` even if there is an onboard RAID controller.

Juanma.
