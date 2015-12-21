---
layout: post
title: Howto set-up a VxVM boot disk from a LVM boot disk
date: 2010-07-20
type: post
published: true
status: publish
categories:
- HP-UX
- Itanium
- Sysadmin
tags:
- EFI
- HP-UX
- Itanium
- LVM
- sysadmin
- systems administration
- vxcp_lvmroot
- VxVM
author: juan_manuel_rey
comments: true
---

Creating a Veritas Volume Manager boot disk using the LVM boot disks as its source probably looks to many as a very complicated process, nothing so far from the reality. In fact the whole conversion can be done with one command, `vxcp_lvmroot`.  In this post I will try to clarify the process and explain some of the underlying mechanisms.

I'm going to take for granted that all of you understand the basic structure of boot disks on Itanium servers. If you have read my [post]({% post_url 2010-06-24-boot-disk-structure-on-integrity-servers %} ) about boot disk structure on Integrity servers you will remember that the disks are composed by three partitions:

-   EFI
-   OS Partition.
-   HPSP - HP Service Partition.

For the purpose of this post the only relevant partition is the OS Partition, also named as `HPUX` in HP-UX hosts.

Unlike LVM, where the volumes are named with numbers (lvol1, lvol2...), in VxVM the volumes follow a specific naming convention that reflects the usage of each one of them:

-   `standvol`
-   `swapvol`
-   `rootvol`
-   `usrvol`
-   `varvol`
-   `tmpvol`
-   `optvol`

Veritas volumes support also a `usetype` field used to provide additional information about the volume to VxVM itself. The three most common use types on HP-UX are:

-   fsgen - File systems and general purpose volumes
-   swap - Swap volumes
-   root - Used for the volume that contains the root file system

The following restrictions must be taken into account for any VxVM boot disk:

-   As in LVM the volumes involved in the boot process (`standvol`, `swapvol` and `rootvol`) must be contiguous.
-   The above volumes can have only one subdisk and can't span to additional disks.
-   The volumes within the root disk can not use dirty region logging (DRL).
-   The private region size is 1MB rather than the default value of 32MB.
-   The `/stand` file system can only be configured with VxFS data layout version 5 or the system will not boot.
-   In PA-RISC systems the `/stand` file system must be HFS, this is necessary because the PA-RISC HP-UX kernel loader is not VxFS-aware.

Following is an example to illustrate the process.

First, with `diskinfo`, verify the size of the current boot disk and the new disk to check that they are the same.

{% highlight text %}
root@robin:/# diskinfo /dev/rdsk/c0t0d0
SCSI describe of /dev/rdsk/c0t0d0:
             vendor: HP      
         product id: Virtual LvDisk  
               type: direct access
               size: 40960000 Kbytes
   bytes per sector: 512
root@robin:/#
root@robin:/# diskinfo /dev/rdsk/c0t8d0
SCSI describe of /dev/rdsk/c0t8d0:
             vendor: HP      
         product id: Virtual LvDisk  
               type: direct access
               size: 40960000 Kbytes
   bytes per sector: 512
root@robin:/#
{% endhighlight %}

After that scrub the new disk, this will prevent possible problems during the creation process because if `vxcp_lvmroot` encounter LVM structures on the disk it will fail.

{% highlight text %}
root@robin:~# dd if=/dev/zero of=/dev/rdsk/c0t8d0 bs=1048576 count=1024  
1024+0 records in
1024+0 records out
root@robin:~#
{% endhighlight %}

Finally launch the `vxcp_lvmroot` command. Before commencing the copy, `vxcp_lvmroot` will determine how many disks are required and will ensure that enough disks have been specified.

Each one of the given disks for the conversion will be checked to make sure that aren't in use as LVM, VxVM or raw disks. Once the appropriate checks have been issued the disks will be given VxVM media names, the disk or disks containing the root will be given `rootdisk##` names and the other disks that are part of the `rootdg` will be given `rootaux##` names, `##` is a number starting on 01.

{% highlight text %}
root@robin:~# /etc/vx/bin/vxcp_lvmroot -v -b c0t8d0
VxVM vxcp_lvmroot INFO V-5-2-4668 10:42: Bootdisk is configured with new-style DSF
VxVM vxcp_lvmroot INFO V-5-2-2499 10:42: Gathering information on LVM root volume group vg00
VxVM vxcp_lvmroot INFO V-5-2-2441 10:42: Checking specified disk(s) for usability
VxVM vxcp_lvmroot INFO V-5-2-4679 10:42: Using legacy-style DSF names
VxVM vxcp_lvmroot INFO V-5-2-2566 10:42: Preparing disk c0t8d0 as a VxVM root disk
VxVM vxcp_lvmroot INFO V-5-2-3767 10:42: Disk c0t8d0 is now EFI partitioned disk c0t8d0s2
VxVM vxcp_lvmroot INFO V-5-2-2537 10:42: Initializing DG rootdg with disk c0t8d0s2 as DM rootdisk01
VxVM vxcp_lvmroot INFO V-5-2-1606 10:42: Copying /dev/vg00/lvol1 (vxfs) to /dev/vx/dsk/rootdg/standvol
VxVM vxcp_lvmroot INFO V-5-2-1604 10:42: Cloning /dev/vg00/lvol2 (swap) to /dev/vx/dsk/rootdg/swapvol
VxVM vxcp_lvmroot INFO V-5-2-1606 10:42: Copying /dev/vg00/lvol3 (vxfs) to /dev/vx/dsk/rootdg/rootvol
VxVM vxcp_lvmroot INFO V-5-2-1606 10:43: Copying /dev/vg00/lvol4 (vxfs) to /dev/vx/dsk/rootdg/homevol
VxVM vxcp_lvmroot INFO V-5-2-1606 10:43: Copying /dev/vg00/lvol5 (vxfs) to /dev/vx/dsk/rootdg/optvol
VxVM vxcp_lvmroot INFO V-5-2-1606 10:50: Copying /dev/vg00/lvol6 (vxfs) to /dev/vx/dsk/rootdg/tmpvol
VxVM vxcp_lvmroot INFO V-5-2-1606 10:50: Copying /dev/vg00/lvol7 (vxfs) to /dev/vx/dsk/rootdg/usrvol
VxVM vxcp_lvmroot INFO V-5-2-1606 10:55: Copying /dev/vg00/lvol8 (vxfs) to /dev/vx/dsk/rootdg/varvol
VxVM vxcp_lvmroot INFO V-5-2-1606 10:58: Copying /dev/vg00/lv_crash (vxfs) to /dev/vx/dsk/rootdg/crashvol
VxVM vxcp_lvmroot INFO V-5-2-4678 10:58: Setting up disk c0t8d0s2 as a boot disk
VxVM vxcp_lvmroot INFO V-5-2-1638 10:59: Installing fstab and fixing dev nodes on new root FS
VxVM vxcp_lvmroot INFO V-5-2-2538 10:59: Installing bootconf & rootconf files in new stand FS
VxVM vxcp_lvmroot INFO V-5-2-2462 10:59: Current setboot values:
VxVM vxcp_lvmroot INFO V-5-2-2569 10:59: Primary:       0/0/0/0.0x0.0x0 /dev/rdisk/disk4
VxVM vxcp_lvmroot INFO V-5-2-2416 10:59: Alternate:      
VxVM vxcp_lvmroot INFO V-5-2-4676 10:59: Making disk /dev/rdisk/disk20_p2 the primary boot disk
VxVM vxcp_lvmroot INFO V-5-2-4663 10:59: Making disk /dev/rdisk/disk4_p2 the alternate boot disk
VxVM vxcp_lvmroot INFO V-5-2-4671 10:59: Disk c0t8d0s2 is now a VxVM rootable boot disk
root@robin:~#
{% endhighlight %}

Now to verify the new VxVM boot disk, first check the newly created `rootdg` diskgroup.

{% highlight text %}
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

dm rootdisk01   c0t8d0s2     auto     1024     40035232 -

v  crashvol     -            ENABLED  ACTIVE   4194304  SELECT    -        fsgen
pl crashvol-01  crashvol     ENABLED  ACTIVE   4194304  CONCAT    -        RW
sd rootdisk01-09 crashvol-01 rootdisk01 28778496 4194304 0        c0t8d0s2 ENA

v  homevol      -            ENABLED  ACTIVE   155648   SELECT    -        fsgen
pl homevol-01   homevol      ENABLED  ACTIVE   155648   CONCAT    -        RW
sd rootdisk01-04 homevol-01  rootdisk01 7077888 155648  0         c0t8d0s2 ENA

v  optvol       -            ENABLED  ACTIVE   9560064  SELECT    -        fsgen
pl optvol-01    optvol       ENABLED  ACTIVE   9560064  CONCAT    -        RW
sd rootdisk01-05 optvol-01   rootdisk01 7233536 9560064 0         c0t8d0s2 ENA

v  rootvol      -            ENABLED  ACTIVE   1048576  SELECT    -        root
pl rootvol-01   rootvol      ENABLED  ACTIVE   1048576  CONCAT    -        RW
sd rootdisk01-03 rootvol-01  rootdisk01 6029312 1048576 0         c0t8d0s2 ENA

v  standvol     -            ENABLED  ACTIVE   1835008  SELECT    -        fsgen
pl standvol-01  standvol     ENABLED  ACTIVE   1835008  CONCAT    -        RW
sd rootdisk01-01 standvol-01 rootdisk01 0      1835008  0         c0t8d0s2 ENA

v  swapvol      -            ENABLED  ACTIVE   4194304  SELECT    -        swap
pl swapvol-01   swapvol      ENABLED  ACTIVE   4194304  CONCAT    -        RW
sd rootdisk01-02 swapvol-01  rootdisk01 1835008 4194304 0         c0t8d0s2 ENA

v  tmpvol       -            ENABLED  ACTIVE   524288   SELECT    -        fsgen
pl tmpvol-01    tmpvol       ENABLED  ACTIVE   524288   CONCAT    -        RW
sd rootdisk01-06 tmpvol-01   rootdisk01 16793600 524288 0         c0t8d0s2 ENA

v  usrvol       -            ENABLED  ACTIVE   6217728  SELECT    -        fsgen
pl usrvol-01    usrvol       ENABLED  ACTIVE   6217728  CONCAT    -        RW
sd rootdisk01-07 usrvol-01   rootdisk01 17317888 6217728 0        c0t8d0s2 ENA

v  varvol       -            ENABLED  ACTIVE   5242880  SELECT    -        fsgen
pl varvol-01    varvol       ENABLED  ACTIVE   5242880  CONCAT    -        RW
sd rootdisk01-08 varvol-01   rootdisk01 23535616 5242880 0        c0t8d0s2 ENA
root@robin:~#
{% endhighlight %}

Verify the contents of the `LABEL` file.

{% highlight text %}
root@robin:~# vxvmboot -v /dev/rdsk/c0t8d0s2

LIF Label File @ (1k) block # 834 on VxVM Disk /dev/rdsk/c0t8d0s2:
Label Entry: 0, Boot Volume start:     3168; length: 1792 MB
Label Entry: 1, Root Volume start:  6032480; length: 1024 MB
Label Entry: 2, Swap Volume start:  1838176; length: 4096 MB
Label Entry: 3, Dump Volume start:  1838176; length: 4096 MB
root@robin:~#
{% endhighlight %}

Check the new boot paths and if everything is OK reboot the server.

{% highlight text %}
root@robin:~# setboot -v
Primary bootpath : 0/0/0/0.0x8.0x0 (/dev/rdisk/disk20)
HA Alternate bootpath :
Alternate bootpath : 0/0/0/0.0x0.0x0 (/dev/rdisk/disk4)

Autoboot is ON (enabled)
setboot: error accessing firmware - Function is not available
The firmware of your system does not support querying or changing the SpeedyBoot
settings.
root@robin:~#
root@robin:~# shutdown -ry 0

SHUTDOWN PROGRAM
06/09/10 11:11:37 WETDST

Broadcast Message from root (console) Wed Jun  9 11:11:37...
SYSTEM BEING BROUGHT DOWN NOW ! ! !

...
{% endhighlight %}

If everything went as expected the server will boot from the new disk and the migration process will be finished.

Juanma.
