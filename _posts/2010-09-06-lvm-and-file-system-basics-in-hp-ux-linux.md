---
title: LVM and file system basics in HP-UX & Linux
date: 2010-09-06
type: post
classes: wide
published: true
status: publish
categories:
- HP-UX
- Linux
- Red Hat
- Sysadmin
tags:
- ext3
- fsadm
- HP-UX
- Linux
- Linux LVM
- LVM
- mkfs.ext3
- sysadmin
- VxFS
author: juan_manuel_rey
comments: true
---

Now that my daily work is more focused on Linux I found myself performing the same basic administration tasks in Linux that I'm used to do in HP-UX. Because of that I thought that a post explaining how the same basic file system and volume management operations are done in both operative systems was necessary, hope you like it.

This is going to be a very basic post intended only as a reference for myself and any other Sysadmin coming from either Linux or HP-UX that wants to know how things are done in the other side. Of course this post is no substitute of the official documentation and the corresponding man pages.

I've used Red Hat Enterprise Linux 5.5 as the Linux version and 11iv3 as the HP-UX version.

The following topics will covered:

-   Volume group creation.
-   Logical volume operations.
-   File system operations.

## Volume group creation

Physical volume and volume group creation are the most basic tasks in LVM, both in Linux and HP-UX, but although command syntax is quite similar in both operative systems the whole process differs in many ways.

### HP-UX

The example used is valid to 11iv2 and 11iv3 HP-UX versions, with the exception of the persistent DSFs you will have to substitute them for the corresponding legacy devices used in 11iv2.

First create the physical volumes.

```
root@hp-ux:/# pvcreate -f /dev/rdisk/disk10
Physical volume "/dev/rdisk/disk10" has been successfully created.
root@hp-ux:/#
root@hp-ux:/# pvcreate -f /dev/rdisk/disk11
Physical volume "/dev/rdisk/disk11" has been successfully created.
root@hp-ux:/#
```

In `/dev` create a directory named as the new volume group, change the ownership to `root:root` and the permissions to `755`.

```
root@hp-ux:/# mkdir -p /dev/vg_new
root@hp-ux:/# chown root:root /dev/vg_new
root@hp-ux:/# chmod 755 /dev/vg_new
```

Go into the VG subdirectory and create the group device special file. For the Linux guys, in HP-UX each volume group must have a group device special file under its subdirectory in `/dev`. This group DSF is created with the `mknod` command, like any other DSFs the group file must have a major and a minor number.

For LVM 1.0 volume groups the major number must be 64 and for the LVM 2.0 one must be 128. Regarding the minor number, the first two digits will uniquely identify the volume group and the remaining digits must be `0000`. In the below example we're creating a 1.0 volume group.

```
root@hp-ux:/dev/vg_new# mknod group c 64 0x010000
```

Change the ownership to `root:sys` and the permissions to `640`.

```
root@hp-ux:/dev/vg_new# chown root:sys group
root@hp-ux:/dev/vg_new# chmod 640 group
```

And create the volume group with the `vgcreate` command, the arguments passed are the two physical volumes previously created and the size in megabytes of the physical extent. The last one is optional and if is not provided the default of 4MB will be automatically set.

```
root@hp-ux:/# vgcreate -s 16 vg_new /dev/disk/disk10 /dev/disk/disk11
Volume group "/dev/vg_new" has been successfully created.
Volume Group configuration for /dev/vg_new has been saved in /etc/lvmconf/vg_new.conf
root@hp-ux:/#
root@hp-ux:/# vgdisplay -v vg_new
--- Volume groups ---
VG Name                     /dev/vg_new
VG Write Access             read/write     
VG Status                   available                 
Max LV                      255    
Cur LV                      0      
Open LV                     0      
Max PV                      16     
Cur PV                      2      
Act PV                      2      
Max PE per PV               6000         
VGDA                        2   
PE Size (Mbytes)            16              
Total PE                    26    
Alloc PE                    0       
Free PE                     26    
Total PVG                   0        
Total Spare PVs             0              
Total Spare PVs in use      0

   --- Physical volumes ---
   PV Name                     /dev/disk/disk10
   PV Status                   available                
   Total PE                    13       
   Free PE                     13       
   Autoswitch                  On        

   PV Name                     /dev/disk/disk11
   PV Status                   available                
   Total PE                    13       
   Free PE                     13       
   Autoswitch                  On     

root@hp-ux:/#
```

### Linux

Create the physical volumes. Here it is where the first difference appears. In HP-UX a physical volume is composed by a whole disk, with the exception of boot disks in Itanium systems, but in Linux a physical volume can be a whole disk or a partition.

For the whole disk the process is pretty much the same as in HP-UX.

```
[root@rhel /]# pvcreate -f /dev/sdb
  Physical volume "/dev/sdb" successfully created
[root@rhel /]# pvdisplay /dev/sdb
  "/dev/sdb" is a new physical volume of "204.00 MB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb
  VG Name               
  PV Size               204.00 MB
  Allocatable           NO
  PE Size (KByte)       0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               Ngyz7I-Z2hL-8R3b-hzA3-qIVc-tZuY-DbCBYn

[root@rhel /]#
```

If you decide to use partitions for the PVs the first, and obvious, thing to do is partition the disk. To setup the disk we'll use the `fdisk` tool, following is an example session:

```
[root@rhel /]# fdisk /dev/sdc

Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4): 1
First cylinder (1-204, default 1):
Using default value 1
Last cylinder or +size or +sizeM or +sizeK (1-204, default 204):
Using default value 204

Command (m for help): t
Selected partition 1
Hex code (type L to list codes): 8e
Changed system type of partition 1 to 8e (Linux LVM)

Command (m for help): p

Disk /dev/sdc: 213 MB, 213909504 bytes
64 heads, 32 sectors/track, 204 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes

   Device Boot      Start         End      Blocks   Id  System
/dev/sdc1               1         204      208880   8e  Linux LVM

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
[root@rhel /]#
```

To explain the session first a new partition is created with the command `n` and the size of the partition is set (in this particular case we are using the whole disk); then we must change the partition type, which by default is set to Linux, to Linux LVM and to do that we use the command `t` and issue `8e` as the corresponding hexadecimal code, the available values for the partition types can be shown  by typing `L`.

```
Command (m for help): t
Selected partition 1
Hex code (type L to list codes): L

 0  Empty           1e  Hidden W95 FAT1 80  Old Minix       bf  Solaris        
 1  FAT12           24  NEC DOS         81  Minix / old Lin c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          82  Linux swap / So c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  83  Linux           c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     84  OS/2 hidden C:  c7  Syrinx         
 5  Extended        41  PPC PReP Boot   85  Linux extended  da  Non-FS data    
 6  FAT16           42  SFS             86  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS       4d  QNX4.x          87  NTFS volume set de  Dell Utility   
 8  AIX             4e  QNX4.x 2nd part 88  Linux plaintext df  BootIt         
 9  AIX bootable    4f  QNX4.x 3rd part 8e  Linux LVM       e1  DOS access     
 a  OS/2 Boot Manag 50  OnTrack DM      93  Amoeba          e3  DOS R/O        
 b  W95 FAT32       51  OnTrack DM6 Aux 94  Amoeba BBT      e4  SpeedStor      
 c  W95 FAT32 (LBA) 52  CP/M            9f  BSD/OS          eb  BeOS fs        
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a0  IBM Thinkpad hi ee  EFI GPT        
 f  W95 Ext'd (LBA) 54  OnTrackDM6      a5  FreeBSD         ef  EFI (FAT-12/16/
10  OPUS            55  EZ-Drive        a6  OpenBSD         f0  Linux/PA-RISC b
11  Hidden FAT12    56  Golden Bow      a7  NeXTSTEP        f1  SpeedStor      
12  Compaq diagnost 5c  Priam Edisk     a8  Darwin UFS      f4  SpeedStor      
14  Hidden FAT16 <3 61  SpeedStor       a9  NetBSD          f2  DOS secondary  
16  Hidden FAT16    63  GNU HURD or Sys ab  Darwin boot     fb  VMware VMFS    
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fc  VMware VMKCORE
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fd  Linux raid auto
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fe  LANstep        
1c  Hidden W95 FAT3 75  PC/IX           be  Solaris boot    ff  BBT            
Hex code (type L to list codes):
```
The changes are written with `w`.

Once the partitions are correctly created, setup the physical volumes.

```
[root@rhel /]# pvcreate -f /dev/sdc1
  Physical volume "/dev/sdc1" successfully created
[root@rhel /]# pvcreate -f /dev/sdd1
  Physical volume "/dev/sdd1" successfully created
[root@rhel /]#
[root@rhel /]# pvs
  PV         VG    Fmt  Attr PSize   PFree  
  /dev/sda2  sysvg lvm2 a-    19.88G      0
  /dev/sdb         lvm2 --   204.00M 204.00M
  /dev/sdc1        lvm2 --   203.98M 203.98M
  /dev/sdd1        lvm2 --   203.98M 203.98M
[root@rhel /]#
```
Now that the PVs are created we can proceed with the volume group creation.

```
[root@rhel /]# vgcreate vg_new /dev/sdc1 /dev/sdd1
 Volume group "vg_new" successfully created
[root@rhel /]# vgdisplay -v vg_new
  Using volume group(s) on command line
  Finding volume group "vg_new"
  /dev/hdc: open failed: No medium found
  --- Volume group ---
  VG Name               vg_new
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               400.00 MB
  PE Size               4.00 MB
  Total PE              100
  Alloc PE / Size       0 / 0   
  Free  PE / Size       100 / 400.00 MB
  VG UUID               lvrrnt-sHbo-eC8j-kC53-Mm5Z-IDDR-RJJtDr

  --- Physical volumes ---
  PV Name               /dev/sdc1     
  PV UUID               kD0jhk-ws8A-ke3L-a7nd-QucS-SAbH-BrmH28
  PV Status             allocatable
  Total PE / Free PE    50 / 50

  PV Name               /dev/sdd1     
  PV UUID               ZP2bLy-FxR3-gYn9-3Dy1-Llgk-6mFI-1iJvTm
  PV Status             allocatable
  Total PE / Free PE    50 / 50

[root@rhel /]#
```

As you could see the process in Linux is slightly simple than in HP-UX.

## Logical volume operations

In this part we will see how to create a logical volume, extend this LV and then remove it from the system.

### HP-UX

The logical volume creation can be done with the `lvcreate` command. With the `-L` option we can specify the size in MB of the new lvol, if `-l` is used instead the size must be provided in logical extents.

```
root@hp-ux:/# lvcreate -n lvol_test -L 256 vg_new
Logical volume "/dev/vg_new/lvol_test_S2" has been successfully created with
character device "/dev/vg_new/rlvol_test_S2".
Logical volume "/dev/vg_new/lvol_test_S2" has been successfully extended.
Volume Group configuration for /dev/vg_new has been saved in /etc/lvmconf/vg_new.conf
root@hp-ux:~# lvdisplay  /dev/vg_new/lvol_test
--- Logical volumes ---
LV Name                     /dev/vg_new/lvol_test
VG Name                     /dev/vg_new
LV Permission               read/write                
LV Status                   available/syncd           
Mirror copies               0            
Consistency Recovery        MWC                 
Schedule                    parallel      
LV Size (Mbytes)            256             
Current LE                  16             
Allocated PE                16             
Stripes                     0       
Stripe Size (Kbytes)        0                   
Bad block                   on         
Allocation                  strict                    
IO Timeout (Seconds)        default             

root@hp-ux:/#
```

Extend a volume. Of course the first prerequisite to extend a volume is to have enough free physical extends in the volume group.

```
root@hp-ux:~# lvextend -L 384 /dev/vg_new/lvol_test
Logical volume "/dev/vg_new/lvol_test" has been successfully extended.
Volume Group configuration for /dev/vg_new has been saved in /etc/lvmconf/vg_new.conf
root@hp-ux:~#
root@hp-ux:~# lvdisplay  /dev/vg_new/lvol_test
--- Logical volumes ---
LV Name                     /dev/vg_new/lvol_test
VG Name                     /dev/vg_new
LV Permission               read/write                
LV Status                   available/syncd           
Mirror copies               0            
Consistency Recovery        MWC                 
Schedule                    parallel      
LV Size (Mbytes)            384             
Current LE                  24             
Allocated PE                24             
Stripes                     0       
Stripe Size (Kbytes)        0                   
Bad block                   on         
Allocation                  strict                    
IO Timeout (Seconds)        default             

root@hp-ux:/#
```

The final step of this part is to remove the logical volume.

```
root@hp-ux:/# lvremove /dev/vg_new/lvol_test
The logical volume "/dev/vg_new/lvol_test" is not empty;
do you really want to delete the logical volume (y/n) : y
Logical volume "/dev/vg_new/lvol_test" has been successfully removed.
Volume Group configuration for /dev/vg_new has been saved in /etc/lvmconf/vg_new.conf
root@hp-ux:/#
```

### Linux

Create the logical volume with the `lvcreate` command, the most basic options (`-L`, `-l`, `-n`) are the same as in HP-UX.

```
[root@rhel /]# lvcreate -n lv_test -L 256 vg_new
  Logical volume "lv_test" created
[root@rhel /]# lvdisplay /dev/vg_new/lv_test
  --- Logical volume ---
  LV Name                /dev/vg_new/lv_test
  VG Name                vg_new
  LV UUID                m5G2vT-dsE1-CycS-BMYR-3MYZ-4y8O-Mx04B8
  LV Write Access        read/write
  LV Status              available
  # open                 0
  LV Size                256.00 MB
  Current LE             16
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:6

[root@rhel /]#
```

Now extend the logical volume to 384 megabytes as we did in HP-UX.

```
[root@rhel /]# lvextend -L 384 /dev/vg_new/lv_test
  Extending logical volume lv_test to 384.00 MB
  Logical volume lv_test successfully resized
[root@rhel /]#
[root@rhel /]# lvdisplay /dev/vg_new/lv_test
  --- Logical volume ---
  LV Name                /dev/vg_new/lv_test
  VG Name                vg_new
  LV UUID                m5G2vT-dsE1-CycS-BMYR-3MYZ-4y8O-Mx04B8
  LV Write Access        read/write
  LV Status              available
  # open                 0
  LV Size                384.00 MB
  Current LE             24
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:6

[root@rhel /]#
```

Remove a volume from the system, like creation and extension is a very straight forward process that can be done with one command.

```
[root@rhel /]# lvremove /dev/vg_new/lv_test
Do you really want to remove active logical volume lv_test? [y/n]: y
  Logical volume "lv_test" successfully removed
[root@rhel /]#
```

Unlike the volume group section, the basic logical operations are performed in almost the same way in both operative systems. Of course if you want to perform mirroring the differences are bigger, but I will leave that for a future post.

## File system operations

The final section of the post is about the basic file system operation, we are going to create a file system on the logical volume of the previous section and later to extend it, including this time the volume group extension.

### HP-UX

Creating the file system with the `newfs` command.

```
root@hp-ux:/# newfs -F vxfs -o largefiles /dev/vg_new/rlvol_test
 version 7 layout
 393216 sectors, 393216 blocks of size 1024, log size 1024 blocks
 largefiles supported
root@hp-ux:/#
```

Create the mount point and mount the filesystem.

```
    root@hp-ux:/# mkdir /data
    root@hp-ux:/# mount /dev/vg_new/lvol_test /data
```

Filesystem extension, in the current section we are going to suppose that the volume group has not enough physical extension to accommodate the new size of the `/data` file system.

After we create a new physical volume in the disk12 we are going to extend the `vg_new` VG.

```
root@hp-ux:/# vgextend vg_new /dev/disk/disk12
Volume group "vg_new" has been successfully extended.
Volume Group configuration for /dev/vg_new has been saved in /etc/lvmconf/vg_new.conf
root@hp-ux:/#
root@hp-ux:/# vgdisplay -v vg_new
--- Volume groups ---
VG Name                     /dev/vg_new
VG Write Access             read/write     
VG Status                   available                 
Max LV                      255    
Cur LV                      0      
Open LV                     0      
Max PV                      16     
Cur PV                      2      
Act PV                      2      
Max PE per PV               6000         
VGDA                        2   
PE Size (Mbytes)            16              
Total PE                    39    
Alloc PE                    24       
Free PE                     15    
Total PVG                   0        
Total Spare PVs             0              
Total Spare PVs in use      0

   --- Logical volumes ---
   LV Name                     /dev/vg_mir/lv_sql
   LV Status                   available/syncd           
   LV Size (Mbytes)            384             
   Current LE                  24             
   Allocated PE                24             
   Used PV                     2

   --- Physical volumes ---
   PV Name                     /dev/disk/disk10
   PV Status                   available                
   Total PE                    13       
   Free PE                     13       
   Autoswitch                  On        

   PV Name                     /dev/disk/disk11
   PV Status                   available                
   Total PE                    13       
   Free PE                     13       
   Autoswitch                  On  

   PV Name                     /dev/disk/disk12
   PV Status                   available                
   Total PE                    13       
   Free PE                     13       
   Autoswitch                  On  

root@hp-ux:/#
```

The next part is to extend the logical volume just as we did in the logical volume operations section.

```
root@hp-ux:/# lvextend -L 512 /dev/vg_new/lvol_test
Logical volume "/dev/vg_new/lvol_test" has been successfully extended.
Volume Group configuration for /dev/vg_new has been saved in /etc/lvmconf/vg_new.conf
root@hp-ux:~#
```

And finally the most creepy part of the part of the process, extending the file system. To be capable of extending a mounted filesystem in HP-UX the OnlineJFS bundle must be installed.

Use the command `fsadm` and with the `-b` option issue the new size in KB as the argument, in the example we want to extend to 512MB, in KB is 524288.

```
root@hp-ux:/# fsadm -F vxfs -b 524288 /data
vxfs fsadm: V-3-23585: /dev/vg00/rlvol5 is currently 7731200 sectors - size will be increased
root#hp-ux:/#
root@hp-ux:/# bdf /data
Filesystem          kbytes    used   avail %used Mounted on
/dev/vg_new/lvol_test
                    524288    5243  524288    1% /data
root@hp-ux:/#
```

### Linux

Here in the filesystem part is where the commands are completely different to HP-UX. In Linux the most common file system types are ext2 and ext3, although others like ext4 or reiserfs are supported.

To create an ext3 file system issue the command `mkfs.ext3` using the logical volume to create the file system on as the argument.

```
[root@rhel ~]# mkfs.ext3 /dev/vg_new/lv_test
mke2fs 1.39 (29-May-2006)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
98304 inodes, 393216 blocks
19660 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=67633152
48 block groups
8192 blocks per group, 8192 fragments per group
2048 inodes per group
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729, 204801, 221185

Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 35 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
[root@rhel ~]#
```

As in HP-UX create the mount point and mount the file system.

```
[root@rhel ~]# mkdir /data
[root@rhel ~]# mount /dev/vg_new/lv_test /data
[root@rhel ~]# df -h /data
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/vg_new-lv_test
                      372M   11M  343M   3% /data
[root@rhel ~]#
```

The final part of the section is the file system extension, as we did in the HP-UX part the first task is to extend the volume group.

```
[root@rhel ~]# vgextend vg_new /dev/sde1
  Volume group "vg_new" successfully extended
[root@rhel ~]# vgdisplay -v vg_new
    Using volume group(s) on command line
    Finding volume group "vg_new"
  --- Volume group ---
  VG Name               vg_new
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  9
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               576.00 MB
  PE Size               16.00 MB
  Total PE              36
  Alloc PE / Size       24 / 384.00 MB
  Free  PE / Size       12 / 192.00 MB
  VG UUID               u32c0h-BPGN-HixT-IzsX-cNnC-EspO-xfweaI

  --- Logical volume ---
  LV Name                /dev/vg_new/lv_test
  VG Name                vg_new
  LV UUID                ZtArMo-Pyyl-BDHX-9CZQ-uEAK-VDqG-t60xy4
  LV Write Access        read/write
  LV Status              available
  # open                 1
  LV Size                384.00 MB
  Current LE             24
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:6

  --- Physical volumes ---
  PV Name               /dev/sdc1     
  PV UUID               kD0jhk-ws8A-ke3L-a7nd-QucS-SAbH-BrmH28
  PV Status             allocatable
  Total PE / Free PE    12 / 0

  PV Name               /dev/sdd1     
  PV UUID               ZP2bLy-FxR3-gYn9-3Dy1-Llgk-6mFI-1iJvTm
  PV Status             allocatable
  Total PE / Free PE    12 / 0

  PV Name               /dev/sde1     
  PV UUID               wbiNu5-csig-uwY7-y14y-3C8Q-oeN0-hAT49g
  PV Status             allocatable
  Total PE / Free PE    12 / 12

[root@rhel ~]#
```

Extend the logical volume with `lvextend`.

```
[root@rhel ~]# lvextend -L 512 /dev/vg_new/lv_test
  Extending logical volume lv_test to 512.00 MB
  Logical volume lv_test successfully resized
[root@rhel ~]# lvs
  LV      VG     Attr   LSize   Origin Snap%  Move Log Copy%  Convert
  lv_home sysvg  -wi-ao 256.00M                                      
  lv_root sysvg  -wi-ao   5.84G                                      
  lv_swap sysvg  -wi-ao   1.00G                                      
  lv_tmp  sysvg  -wi-ao   1.00G                                      
  lv_usr  sysvg  -wi-ao   9.75G                                      
  lv_var  sysvg  -wi-ao   2.03G                                      
  lv_test vg_new -wi-ao 512.00M                                      
[root@rhel ~]#
```

Finally resize the file system, to do that use the `resize2fs` tool. Unlike in HP-UX with `fsadm`, that needs the new size as an argument  in order to extend the file system, if you simply issue the logical volume as the argument the `resize2fs` utility will extend the file system to the maximum size available in the LV.

```
[root@rhel ~]# resize2fs /dev/vg_new/lv_test
resize2fs 1.39 (29-May-2006)
Filesystem at /dev/vg_new/lv_test is mounted on /data; on-line resizing required
Performing an on-line resize of /dev/vg_new/lv_test to 524288 (1k) blocks.
The filesystem on /dev/vg_new/lv_test is now 524288 blocks long.

[root@rhel ~]#
```

And at this point we are done. Courteous comments are welcome as always.

Juanma.
