---
layout: post
title: Coming back to the IVM world
date: 2010-03-02
type: post
published: true
status: publish
categories:
- HP-UX
- Itanium
- Virtualization
tags:
- 11iv2
- 11iv3
- HP-UX
- HPVM
- Integrity VMs
- LVM
author: juan_manuel_rey
comments: true
---

Yes I have to admit it, it's been a while since the last time I created an Integrity Virtual Machine. In my last job didn't have HPVM and here the VMs were already running when I arrived. So a few weeks ago I decided to cut my teeth again with HPVM, specially since I am pushing forward very hard for an OS and HPVM version upgrade of the IVM cluster which is currently running HP-UX 11.23 with HPVM 3.5.

First logical step in order to get proficient again with IVM is to create a new virtual machine. I asked Javi, our storage guy, for a new LUN and after add it to my lab server I started the whole process.

Some of the steps are obvious for any HP-UX Sysadmin, like VGs and LVs creation, but I decided to show the commands in order to maintain some consistency across this how-to/checklist/what-ever-you-like-to-call-it.

-   Create a volume group for the IVM virtual disks.

```
[root@hpvmhost] ~ # vgcreate -s 16 -e 6000 vg_vmtest /dev/dsk/c15t7d1
Volume group "/dev/vg_vmtest" has been successfully created.
Volume Group configuration for /dev/vg_vmtest has been saved in /etc/lvmconf/vg_vmtest.conf
[root@hpvmhost] ~ #
[root@hpvmhost] ~ # vgextend vg_vmtest /dev/dsk/c5t7d1  /dev/dsk/c7t7d1 /dev/dsk/c13t7d1
Volume group "vg_vmtest" has been successfully extended.
Volume Group configuration for /dev/vg_vmtest has been saved in /etc/lvmconf/vg_vmtest.conf
[root@hpvmhost] ~ # vgdisplay -v vg_vmtest
--- Volume groups ---
VG Name                     /dev/vg_vmtest
VG Write Access             read/write     
VG Status                   available                 
Max LV                      255    
Cur LV                      0      
Open LV                     0      
Max PV                      16     
Cur PV                      1      
Act PV                      1      
Max PE per PV               6000         
VGDA                        2   
PE Size (Mbytes)            16              
Total PE                    3199    
Alloc PE                    0       
Free PE                     3199    
Total PVG                   0        
Total Spare PVs             0              
Total Spare PVs in use      0                     

--- Physical volumes ---
PV Name                     /dev/dsk/c15t7d1
PV Name                     /dev/dsk/c5t7d1  Alternate Link
PV Name                     /dev/dsk/c7t7d1  Alternate Link
PV Name                     /dev/dsk/c13t7d1 Alternate Link
PV Status                   available                
Total PE                    3199    
Free PE                     3199    
Autoswitch                  On        
Proactive Polling           On               

[root@hpvmhost] ~ #
```

-   Create one `lvol` for each disk you want to add to your virtual machine, of course these logical volumes must belong to the volume group previously created.

```
[root@hpvmhost] ~ # lvcreate -L 12000 -n ivm1d1 vg_vmtest
Logical volume "/dev/vg_vmtest/ivm1d1" has been successfully created with
character device "/dev/vg_vmtest/rivm1d1".
Logical volume "/dev/vg_vmtest/ivm1d1" has been successfully extended.
Volume Group configuration for /dev/vg_vmtest has been saved in /etc/lvmconf/vg_vmtest.conf
[root@hpvmhost] ~ #
[root@hpvmhost] ~ # lvcreate -L 12000 -n ivm1d2 vg_vmtest
Logical volume "/dev/vg_vmtest/ivm1d2" has been successfully created with
character device "/dev/vg_vmtest/rivm1d2".
Logical volume "/dev/vg_vmtest/ivm1d2" has been successfully extended.
Volume Group configuration for /dev/vg_vmtest has been saved in /etc/lvmconf/vg_vmtest.conf
[root@hpvmhost] ~ #
```

-   Now we're going to do some real stuff. Create the IVM with the `hpvmcreate` command and use the `hpvmstatus` to check that everything went well :

```
[root@hpvmhost] ~ # hpvmcreate -P ivm1 -O hpux  
[root@hpvmhost] ~ # hpvmstatus -P ivm1
[Virtual Machine Details]
Virtual Machine Name VM #  OS Type State
==================== ===== ======= ========
ivm1                     8 HPUX    Off       

[Authorized Administrators]
Oper Groups:  
Admin Groups:
Oper Users:   
Admin Users:  

[Virtual CPU Details]
#vCPUs Entitlement Maximum
====== =========== =======
 1       10.0%  100.0%

[Memory Details]
Total    Reserved
Memory   Memory  
=======  ========
 2 GB     64 MB

[Storage Interface Details]
Guest                                 Physical
Device  Adaptor    Bus Dev Ftn Tgt Lun Storage   Device
======= ========== === === === === === ========= =========================

[Network Interface Details]
Interface Adaptor    Name/Num   PortNum Bus Dev Ftn Mac Address
========= ========== ========== ======= === === === =================

[Misc Interface Details]
Guest                                 Physical
Device  Adaptor    Bus Dev Ftn Tgt Lun Storage   Device
======= ========== === === === === === ========= =========================
serial  com1                           tty       console
[root@hpvmhost] ~ #
```

We have a new virtual machine created but with no resources at all.

If you have read the [HPVM documentation](http://docs.hp.com/en/T2767-90180/index.html), and you should, probably know that every resource can be assigned at this step but I like to add them later one by one.

Since now we're going to use the *hpvmstatus* to verify every change made. This command can be invoked without options to show a general summary or can query a single virtual machine, a verbose option is also available with `-V`. Take a look of its man page to check more options.

-   Add more CPU and RAM. The default values are 1 vCPU and 2GB of RAM, more can be assigned with `hpvmmodify`:

```
[root@hpvmhost] ~ # hpvmmodify -P ivm1 -c 2
[root@hpvmhost] ~ # hpvmmodify -P ivm1 -r 4G
[root@hpvmhost] ~ # hpvmstatus
[Virtual Machines]
Virtual Machine Name VM #  OS Type State     #VCPUs #Devs #Nets Memory  Runsysid
==================== ===== ======= ========= ====== ===== ===== ======= ========
oratest01                1 HPUX    On (OS)        4    10     3   16 GB        0
oratest02                2 HPUX    On (OS)        4     8     3   16 GB        0
sapvm01                  3 HPUX    Off            3     8     3    8 GB        0
sapvm02                  4 HPUX    Off            3     7     3    8 GB        0
sles01                   5 LINUX   On (OS)        1     4     3    4 GB        0
rhel01                   6 LINUX   Off            1     4     3    4 GB        0
hp-vxvm                  7 HPUX    On (OS)        2    17     3    6 GB        0
ivm1                     8 HPUX    Off            2     0     0    4 GB        0
[root@hpvmhost] ~ #
```

-   With the CPUs and RAM finished it's time to add the storage devices, as always we're going to use `hpvmmodify`:

```
[root@hpvmhost] ~ # hpvmmodify -P ivm1 -a disk:scsi::lv:/dev/vg_vmtest/rivm1d1
[root@hpvmhost] ~ # hpvmmodify -P ivm1 -a disk:scsi::lv:/dev/vg_vmtest/rivm1d2
[root@hpvmhost] ~ # hpvmmodify -P ivm1 -a dvd:scsi::disk:/dev/rdsk/c1t4d0
[root@hpvmhost] ~ # hpvmstatus -P ivm1
[Virtual Machine Details]
Virtual Machine Name VM #  OS Type State
==================== ===== ======= ========
ivm1                     8 HPUX    Off       

[Authorized Administrators]
Oper Groups:  
Admin Groups:
Oper Users:   
Admin Users:  

[Virtual CPU Details]
#vCPUs Entitlement Maximum
====== =========== =======
 2       10.0%  100.0%

[Memory Details]
Total    Reserved
Memory   Memory  
=======  ========
 4 GB     64 MB

[Storage Interface Details]
Guest                                 Physical
Device  Adaptor    Bus Dev Ftn Tgt Lun Storage   Device
======= ========== === === === === === ========= =========================
disk    scsi         0   2   0   0   0 lv        /dev/vg_vmtest/rivm1d1
disk    scsi         0   2   0   1   0 lv        /dev/vg_vmtest/rivm1d2
dvd     scsi         0   2   0   2   0 disk      /dev/rdsk/c1t4d0

[Network Interface Details]
Interface Adaptor    Name/Num   PortNum Bus Dev Ftn Mac Address
========= ========== ========== ======= === === === =================

[Misc Interface Details]
Guest                                 Physical
Device  Adaptor    Bus Dev Ftn Tgt Lun Storage   Device
======= ========== === === === === === ========= =========================
serial  com1                           tty       console
[root@hpvmhost] ~ #
```

An important tip about the storage devices, remember that you have to use the character device file of the LV. If a block device is used you will get the following error:

```
[root@hpvmhost] ~ # hpvmmodify -P ivm1 -a disk:scsi::lv:/dev/vg_vmtest/ivm1d1
hpvmmodify: WARNING (ivm1): Expecting a character device file for disk backing file, but '/dev/vg_vmtest/ivm1d1' appears to be a block device.
hpvmmodify: ERROR (ivm1): Illegal blk device '/dev/vg_vmtest/ivm1d1' as backing device.
hpvmmodify: ERROR (ivm1): Unable to add device '/dev/vg_vmtest/ivm1d1'.
hpvmmodify: Unable to create device disk:scsi::lv:/dev/vg_vmtest/ivm1d1.
hpvmmodify: Unable to modify the guest.
[root@hpvmhost] ~ #
```

-   Virtual networking 1: First check the available virtual switches with `hpvmnet`:

```
[root@hpvmhost] / # hpvmnet
Name     Number State   Mode      NamePPA  MAC Address    IP Address
======== ====== ======= ========= ======== ============== ===============
localnet      1 Up      Shared             N/A            N/A
vlan02        2 Up      Shared    lan3     0x000000000000 192.168.1.12
vlan03        3 Up      Shared    lan4     0x001111111111 10.10.3.4
[root@hpvmhost] / #
```

-   Virtual Networking 2: Add a couple of `vnics` to the virtual machine.

```
[root@hpvmhost] / # hpvmmodify -P ivm1 -a network:lan:vswitch:vlan02
[root@hpvmhost] / # hpvmmodify -P ivm1 -a network:lan:vswitch:localnet
[root@hpvmhost] / #
[root@hpvmhost] / # hpvmstatus -P ivm1
[Virtual Machine Details]
Virtual Machine Name VM #  OS Type State
==================== ===== ======= ========
ivm1                     8 HPUX    Off       

[Authorized Administrators]
Oper Groups:  
Admin Groups:
Oper Users:   
Admin Users:  

[Virtual CPU Details]
#vCPUs Entitlement Maximum
====== =========== =======
 2       10.0%  100.0%

[Memory Details]
Total    Reserved
Memory   Memory  
=======  ========
 4 GB     64 MB

[Storage Interface Details]
Guest                                 Physical
Device  Adaptor    Bus Dev Ftn Tgt Lun Storage   Device
======= ========== === === === === === ========= =========================
disk    scsi         0   2   0   0   0 lv        /dev/vg_vmtest/rivm1d1
disk    scsi         0   2   0   1   0 lv        /dev/vg_vmtest/rivm1d2
dvd     scsi         0   2   0   2   0 disk      /dev/rdsk/c1t4d0

[Network Interface Details]
Interface Adaptor    Name/Num   PortNum Bus Dev Ftn Mac Address
========= ========== ========== ======= === === === =================
vswitch   lan        vlan02     8         0   0   0 56-e9-e3-09-6a-22
vswitch   lan        localnet   8         0   1   0 ae-d6-f7-fa-4e-3e

[Misc Interface Details]
Guest                                 Physical
Device  Adaptor    Bus Dev Ftn Tgt Lun Storage   Device
======= ========== === === === === === ========= =========================
serial  com1                           tty       console
[root@hpvmhost] / #
```

-   And we have an IVM ready to be used. To start it use the  `hpvmstart` command and access its console with `hpvmconsole`, the interface is almost equal to GSP/MP.

```
[root@hpvmhost] ~ # hpvmstart -P ivm1
(C) Copyright 2000 - 2008 Hewlett-Packard Development Company, L.P.
Opening minor device and creating guest machine container
Creation of VM, minor device 3
Allocating guest memory: 4096MB
  allocating low RAM (0-80000000, 2048MB)
/opt/hpvm/lbin/hpvmapp (/var/opt/hpvm/uuids/2b3b1198-2062-11df-9e06-001a4b07f002/vmm_config.current): Allocated 2147483648 bytes at 0x6000000100000000
  allocating high RAM (100000000-180000000, 2048MB)
/opt/hpvm/lbin/hpvmapp (/var/opt/hpvm/uuids/2b3b1198-2062-11df-9e06-001a4b07f002/vmm_config.current): Allocated 2147483648 bytes at 0x6000000200000000
    locking memory: 100000000-180000000
    allocating datalogger memory: FF800000-FF840000 (256KB for 155KB)
/opt/hpvm/lbin/hpvmapp (/var/opt/hpvm/uuids/2b3b1198-2062-11df-9e06-001a4b07f002/vmm_config.current): Allocated 262144 bytes at 0x6000000300000000
    locking datalogger memory
  allocating firmware RAM (fff00000-fff20000, 128KB)
/opt/hpvm/lbin/hpvmapp (/var/opt/hpvm/uuids/2b3b1198-2062-11df-9e06-001a4b07f002/vmm_config.current): Allocated 131072 bytes at 0x6000000300080000
    locked SAL RAM: 00000000fff00000 (8KB)
    locked ESI RAM: 00000000fff02000 (8KB)
    locked PAL RAM: 00000000fff04000 (8KB)
    locked Min Save State: 00000000fff06000 (8KB)
    locked datalogger: 00000000ff800000 (256KB)
Loading boot image
Image initial IP=102000 GP=67E000
Initialize guest memory mapping tables
Starting event polling thread
Starting thread initialization
Daemonizing....
hpvmstart: Successful start initiation of guest 'ivm1'
[root@hpvmhost] ~ #
[root@hpvmhost] ~ # hpvmconsole -P ivm1

   vMP MAIN MENU

         CO: Console
         CM: Command Menu
         CL: Console Log
         SL: Show Event Logs
         VM: Virtual Machine Menu
         HE: Main Help Menu
         X: Exit Connection

[ivm1] vMP> co

       (Use Ctrl-B to return to vMP main menu.)

- - - - - - - - - - Prior Console Output - - - - - - - - - -
```

And we are finished. I'm not going through the installation process since it's not the objective of this post and it is very well documented in the HP-UX documentation.

I really enjoyed this post, it has been very useful exercise in order to re-learn the roots of HPVM and a very good starting point for the HP-UX/HPVM upgrade I'm going to undertake during next weeks.

Juanma.
