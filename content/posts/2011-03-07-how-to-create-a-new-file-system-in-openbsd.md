---
title: How to create a new file system in OpenBSD
date: 2011-03-07
tags:
- bsd
- sysadmin
showComments: true
---

Another self-reference short post that can be useful for any [OpenBSD](http://www.openbsd.org "OpenBSD") newbie and because I love to go through the basics from time to time just to don't lose my expertise.

Also I'll explain the basics about how disks and partitions are managed in OpenBSD

## Identify and prepare the disk

There are two types of disks common to most platform where OpenBSD can run:

- IDE disks. Identified as `wdX`
- SCSI disks and devices that use SCSI commands. Identified as `sdX`. This category includes also the USB and SATA disks.

Our disk for this example will be a SCSI one, sd3. Using `dmesg` check that the system has recognized the device.

```text
[root@obsd ~]# dmesg |grep SCSI
sd0 at scsibus1 targ 0 lun 0: <VMware,, VMware Virtual S, 1.0> SCSI2 0/direct fixed
sd1 at scsibus1 targ 1 lun 0: <VMware,, VMware Virtual S, 1.0> SCSI2 0/direct fixed
sd2 at scsibus1 targ 2 lun 0: <VMware,, VMware Virtual S, 1.0> SCSI2 0/direct fixed
sd3 at scsibus1 targ 3 lun 0: <VMware,, VMware Virtual S, 1.0> SCSI2 0/direct fixed
[root@obsd ~]#
```

Next initialize the MBR of the disk using the default template. Use `fdisk -i` to perform the task.

```text
[root@obsd ~]# fdisk -i sd3
Do you wish to write new MBR and partition table? [n] y
Writing MBR at offset 0.
[root@obsd ~]# fdisk sd3
Disk: sd3       geometry: 1044/255/63 [16777216 Sectors]
Offset: 0       Signature: 0xAA55
            Starting         Ending         LBA Info:
 #: id      C   H   S -      C   H   S [       start:        size ]
-------------------------------------------------------------------------------
 0: 00      0   0   0 -      0   0   0 [           0:           0 ] unused     
 1: 00      0   0   0 -      0   0   0 [           0:           0 ] unused     
 2: 00      0   0   0 -      0   0   0 [           0:           0 ] unused     
*3: A6      0   1   2 -   1043 254  63 [          64:    16771796 ] OpenBSD    
[root@obsd ~]#
```

As it can be seen the partition number 3 has been initialize as OpenBSD. In the example we are using the whole disk for OpenBSD, if not you should enter in edition mode with `fdisk -i <disk>` and partition the disk appropriately.

## Partition the disk using `disklabel`

Here you can be confused, because really we are going to partition the partition. These partition are also known as **Filesystem Partitions** since is on top of them where the filesystems and swap devices are created. And the `fdisk` partitions are known as **MBR Partitions**.

In other BSD systems, like FreeBSD, these partitions are called slices however in OpenBSD historically have been called also partitions which have lead to some confusion.

`disklabel` partitions are identified by appending a letter to the disk identifier, like `sd3a` which represents the first partition of the SCSI disk 3. There are some reserved letters:

- `a` - represents always the root partition of the disk.
- `b` - is always use a swap device.
- `c` - represents the whole disk.

And finally here it is an example on how to create a filesystem partition. Use `disklabel -E <disk>` to edit the disk.

```text
[root@obsd ~]# disklabel -E sd3
Label editor (enter '?' for help at any prompt)
> p
OpenBSD area: 64-16771860; size: 16771796; free: 16771796
#                size           offset  fstype [fsize bsize  cpg]
  c:         16777216                0  unused                  
> a a
offset: [64]
size: [16771796]
FS type: [4.2BSD]
> p
OpenBSD area: 64-16771860; size: 16771796; free: 20
#                size           offset  fstype [fsize bsize  cpg]
  a:         16771776               64  4.2BSD   2048 16384    1
  c:         16777216                0  unused                  
> q
Write new label?: [y] y
[root@obsd ~]#
[root@obsd ~]# disklabel sd3
# /dev/rsd3c:
type: SCSI
disk: SCSI disk
label: VMware Virtual S
uid: 2462f082e9092afc
flags:
bytes/sector: 512
sectors/track: 63
tracks/cylinder: 255
sectors/cylinder: 16065
cylinders: 1044
total sectors: 16777216
boundstart: 64
boundend: 16771860
drivedata: 0

16 partitions:
#                size           offset  fstype [fsize bsize  cpg]
  a:         16771776               64  4.2BSD   2048 16384    1
  c:         16777216                0  unused                  
[root@obsd ~]#
```

## Create the file system

Use `newfs` against the special raw device file to create the file system. By default OpenBSD uses the 4.3BSD file system to build file systems with backward compatibility with older boot ROMS, however it also support Fast File System (FFS) as the default format for filesystem smaller that 1TB and Enhanced Fast File System (FFS2) for file systems larger than 1TB.

```text
[root@obsd ~]# newfs /dev/rsd3a
/dev/rsd3a: 8192.0MB in 16777216 sectors of 512 bytes
41 cylinder groups of 202.47MB, 12958 blocks, 25984 inodes each
super-block backups (for fsck -b #) at:
 32, 414688, 829344, 1244000, 1658656, 2073312, 2487968, 2902624, 3317280, 3731936, 4146592, 4561248, 4975904, 5390560, 5805216, 6219872, 6634528, 7049184, 7463840, 7878496, 8293152, 8707808, 9122464,
 9537120, 9951776, 10366432, 10781088, 11195744, 11610400, 12025056, 12439712, 12854368, 13269024, 13683680, 14098336, 14512992, 14927648, 15342304, 15756960, 16171616, 16586272,
[root@obsd ~]#
```

Next you can mount your newly created file system like any other Unix system.

```text
[root@obsd ~]# mkdir /data
[root@obsd ~]# mount /dev/sd3a /data
[root@obsd ~]#
[root@obsd ~]# df -h
Filesystem     Size    Used   Avail Capacity  Mounted on
/dev/sd0a      5.8G   43.1M    5.4G     1%    /
/dev/sd0d      508M   10.4M    472M     2%    /home
/dev/sd0f     1001M    6.0K    951M     0%    /tmp
/dev/sd0g     25.3G    625M   23.4G     3%    /usr
/dev/sd0e      1.9G    5.1M    1.8G     0%    /var
/dev/sd3a      7.9G    2.0K    7.5G     0%    /data
[root@obsd ~]#
```

And of course you can add it to the `/etc/fstab` file to provide persistence through a reboot of the system.

Juanma.
