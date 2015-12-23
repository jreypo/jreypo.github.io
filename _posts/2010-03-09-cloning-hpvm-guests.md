---
layout: post
title: Cloning HPVM guests
date: 2010-03-09
type: post
published: true
status: publish
categories:
- HP-UX
- Itanium
- Sysadmin
- Virtualization
tags:
- Dynamic Root Disk
- HP-UX
- HPVM
- hpvmclone
- Integrity VMs
- Itanium
- sysadmin
- systems administration
author: juan_manuel_rey
comments: true
---

Our next step in the wonderful HPVM World is... cloning virtual machines.

If you have used VMware Virtual Infrastructure cloning, probably are used to the easy *right-click and clone vm* procedure. Sadly HPVM cloning has nothing in common with it. In fact the process to clone a virtual machine can be a little creepy.

Of course there is a `hpvmclone` command and anyone can think, as I did the first time I had to clone an IVM, I only have to provide the source VM, the new VM name and *voilà* everything will be done:

{% highlight text %}
[root@hpvmhost] ~ # hpvmclone -P ivm1 -N ivm_clone01
[root@hpvmhost] ~ # hpvmstatus
[Virtual Machines]
Virtual Machine Name VM #  OS Type State     #VCPUs #Devs #Nets Memory  Runsysid
==================== ===== ======= ========= ====== ===== ===== ======= ========
ivm1                     9 HPUX    Off            3     3     2    2 GB        0
ivm2                    10 HPUX    Off            1     7     1    3 GB        0
ivm_clone01             11 HPUX    Off            3     3     2    2 GB        0
[root@hpvmhost] ~ #
{% endhighlight %}

The new virtual machine can be seen and everything seems to be fine but when you ask for the configuration details of the new IVM a nasty surprise will appear... the storage devices had not been cloned instead it looks that `hpvmclone` simply mapped the devices of the source IVM to the new IVM:

{% highlight text %}
[root@hpvmhost] ~ # hpvmstatus -P ivm_clone01
[Virtual Machine Details]
Virtual Machine Name VM #  OS Type State
==================== ===== ======= ========
ivm_clone01             11 HPUX    Off      

[Authorized Administrators]
Oper Groups: 
Admin Groups:
Oper Users:  
Admin Users: 

[Virtual CPU Details]
#vCPUs Entitlement Maximum
====== =========== =======
     3       20.0%  100.0%

[Memory Details]
Total    Reserved
Memory   Memory 
=======  ========
   2 GB     64 MB

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
vswitch   lan        vlan02     11        0   0   0 f6-fb-bf-41-78-63
vswitch   lan        localnet   10        0   1   0 2a-69-35-d5-c1-5f

[Misc Interface Details]
Guest                                 Physical
Device  Adaptor    Bus Dev Ftn Tgt Lun Storage   Device
======= ========== === === === === === ========= =========================
serial  com1                           tty       console
[root@hpvmhost] ~ #
{% endhighlight %}

With this configuration the virtual machines can't be booted at the same time. So, what is the purpose of `hpvmclone` if the newly cloned node can't be used simultaneously with the original? Honestly this makes no sense at least for me.

At that point and since I really wanted to use both machines in a test cluster I decided to do a little research through Google and [ITRC](http://www.itrc.hp.com/ "IT Resource Center").

After reading again the official documentation, a few dozens posts regarding HPVM cloning and HPVM in general and a few very nice posts in Daniel Parkes' [HP-UX Tips & Tricks](http://www.hpuxtips.es/) site I finally came up with three different methods to successfully and *physically* clone an Integrity Virtual Machine.

### METHOD 1: Using `dd`

-   Create the LVM structure for the new virtual machine on the host.
-   Use `dd` to copy every storage device from the source virtual machine.

{% highlight text %}
[root@hpvmhost] ~ # dd if=/dev/vg_vmtest/rivm1d1 of=/dev/vg_vmtest/rclone01_d1 bs=1024k
12000+0 records in
12000+0 records out
[root@hpvmhost] ~ #
[root@hpvmhost] ~ # dd if=/dev/vg_vmtest/rivm1d2 of=/dev/vg_vmtest/rclone01_d2 bs=1024k
12000+0 records in
12000+0 records out
[root@hpvmhost] ~ #
{% endhighlight %}

-   Using `hpvmclone` create the new machine and in the same command add the new storage devices and delete the old ones from its configuration, any resource can also be modified at this point like with `hpvmcreate`.

{% highlight text %}
[root@hpvmhost] ~ # hpvmclone -P ivm1 -N clone01 -d disk:scsi:0,2,0:lv:/dev/vg_vmtest/rivm1d1 \
> -d disk:scsi:0,2,1:lv:/dev/vg_vmtest/rivm1d2 \
> -a disk:scsi::lv:/dev/vg_vmtest/rclone01_d1 \
> -a disk:scsi::lv:/dev/vg_vmtest/rclone01_d2 \
> -l "Clone-cluster 01" \
> -B manual
[root@hpvmhost] ~ #
{% endhighlight %}

-   Start the new virtual machine and make the necessary changes to the guest OS (network, hostname, etc).

### METHOD 2: Clone the virtual storage devices at the same time the IVM is cloned.

Yes, yes and yes it can be done with `hpvmclone`, you have to use the `-b` switch and provide the storage resource to use.

I really didn't test this procedure with other devices apart from the booting disk/disks. In theory the man page of the command and the HPVM documentation states that this option can be used to specify the booting device of the clone but I used to clone a virtual machine with one boot disk and one with two disks and in both cases it worked without problems.

-   As in METHOD 1 create the necessary LVM infrastructure for the new IVM.
-   Once the lvols are created clone the virtual machine.

{% highlight text %}
[root@hpvmhost] ~ # hpvmclone -P ivm1 -N vxcl01 -a disk:scsi::lv:/dev/vg_vmtest/rvxcl01_d1 \
> -a disk:scsi::lv:/dev/vg_vmtest/rvxcl01_d2 \
> -b disk:scsi:0,2,0:lv:/dev/vg_vmtest/rvxcl01_d1 \
> -b disk:scsi:0,2,1:lv:/dev/vg_vmtest/rvxcl01_d2 \
> -d disk:scsi:0,2,0:lv:/dev/vg_vmtest/rivm1d1 \
> -d disk:scsi:0,2,1:lv:/dev/vg_vmtest/rivm1d2 \
> -B manual
12000+0 records in
12000+0 records out
hpvmclone: Virtual storage cloned successfully.
12000+0 records in
12000+0 records out
hpvmclone: Virtual storage cloned successfully.
[root@hpvmhost] ~ #
{% endhighlight %}

-   Start the virtual machine.
-   Now log into the virtual machine to check the start-up process and to make any change needed.

### METHOD 3: Dynamic Root Disk.

Since with DRD a clone of `vg00` can be produced we can use it too to clone an Integrity Virtual Machine.

-   First step is to create a new `lvol` that will contain the clone of `vg00`, it has to be at least the same size as the original disk.
-   Install the last DRD version supported on the virtual machin to clone.
-   Add the new volume to the source virtual machine and from the guest OS re-scan for the new disk.
-   Now proceed with the DRD clone.

{% highlight text %}
root@ivm2:~# drd clone -v -x overwrite=true -t /dev/disk/disk15   

=======  03/09/10 15:45:15 MST  BEGIN Clone System Image (user=root)  (jobid=ivm2)

       * Reading Current System Information
       * Selecting System Image To Clone
       * Converting legacy Dsf "/dev/dsk/c0t0d0" to "/dev/disk/disk3"
       * Selecting Target Disk
NOTE:    There may be LVM 2 volumes configured that will not be recognized.
       * Selecting Volume Manager For New System Image
       * Analyzing For System Image Cloning
       * Creating New File Systems
       * Copying File Systems To New System Image
       * Copying File Systems To New System Image succeeded.
       * Making New System Image Bootable
       * Unmounting New System Image Clone
       * System image: "sysimage_001" on disk "/dev/disk/disk15"

=======  03/09/10 16:05:20 MST  END Clone System Image succeeded. (user=root)  (jobid=ivm2)

root@ivm2:~#
{% endhighlight %}

-   Mount the new image.

{% highlight text %}
root@ivm2:~# drd mount -v

=======  03/09/10 16:09:08 MST  BEGIN Mount Inactive System Image (user=root)  (jobid=ivm2)

       * Checking for Valid Inactive System Image
       * Locating Inactive System Image
       * Preparing To Mount Inactive System Image
       * Selected inactive system image "sysimage_001" on disk "/dev/disk/disk15".
       * Mounting Inactive System Image
       * System image: "sysimage_001" on disk "/dev/disk/disk15"

=======  03/09/10 16:09:26 MST  END Mount Inactive System Image succeeded. (user=root)  (jobid=ivm2)

root@ivm2:~#
{% endhighlight %}

-   On the mounted image edit the `netconf` file and modify the hostname to "" and remove any network configuration such as IP address, gateway, etc. The image is mounted on `/var/opt/drd/mnts/sysimage_001`.
-   Move or delete the DRD XML registry file in `/var/opt/drd/mnts/sysimage_001/var/opt/drd/registry` in order to avoid any problems during the boot of the clone since the source disk will not be present.
-   Unmount the image.

{% highlight text %}
root@ivm2:~# drd umount -v

=======  03/09/10 16:20:45 MST  BEGIN Unmount Inactive System Image (user=root)  (jobid=ivm2)

       * Checking for Valid Inactive System Image
       * Locating Inactive System Image
       * Preparing To Unmount Inactive System Image
       * Unmounting Inactive System Image
       * System image: "sysimage_001" on disk "/dev/disk/disk15"

=======  03/09/10 16:20:58 MST  END Unmount Inactive System Image succeeded. (user=root)  (jobid=ivm2)

root@ivm2:~#
{% endhighlight %}

-   Now we are going to create the new virtual machine with `hpvmclone`. Of course the new IVM can be created through `hpvmcreate` and add the new disk as its boot disk.

{% highlight text %}
[root@hpvmhost] ~ # hpvmclone -P ivm2 -N ivm3 -B manual -d disk:scsi:0,1,0:lv:/dev/vg_vmtest/rivm2disk
[root@hpvmhost] ~ # hpvmstatus -P ivm3
[Virtual Machine Details]
Virtual Machine Name VM #  OS Type State
==================== ===== ======= ========
ivm3                     4 HPUX    Off      

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
   3 GB     64 MB

[Storage Interface Details]
Guest                                 Physical
Device  Adaptor    Bus Dev Ftn Tgt Lun Storage   Device
======= ========== === === === === === ========= =========================
dvd     scsi         0   1   0   1   0 disk      /dev/rdsk/c1t4d0
disk    scsi         0   1   0   2   0 lv        /dev/vg_vmtest/rivm3disk

[Network Interface Details]
Interface Adaptor    Name/Num   PortNum Bus Dev Ftn Mac Address
========= ========== ========== ======= === === === =================
vswitch   lan        vlan02     11        0   0   0 52-4f-f9-5e-02-82

[Misc Interface Details]
Guest                                 Physical
Device  Adaptor    Bus Dev Ftn Tgt Lun Storage   Device
======= ========== === === === === === ========= =========================
serial  com1                           tty       console
[root@hpvmhost] ~ #
{% endhighlight %}

-   Final step is to boot the newly create machine, from the EFI menu we're going to create a new boot file.
-   First select the *Boot option maintenance menu*:

{% highlight text %}
EFI Boot Manager ver 1.10 [14.62] [Build: Mon Oct  1 09:27:26 2007]

Please select a boot option

    HP-UX Primary Boot: 0/0/1/0.0.0                                
    EFI Shell [Built-in]                                           
    Boot option maintenance menu                                   

    Use ^ and v to change option(s). Use Enter to select an option
{% endhighlight %}

-   Now go to `Add a Boot Option`.

{% highlight text %}
EFI Boot Maintenance Manager ver 1.10 [14.62]

Main Menu. Select an Operation

        Boot from a File                                           
        Add a Boot Option                                          
        Delete Boot Option(s)                                      
        Change Boot Order                                          

        Manage BootNext setting                                    
        Set Auto Boot TimeOut                                      

        Select Active Console Output Devices                       
        Select Active Console Input Devices                        
        Select Active Standard Error Devices                       

        Cold Reset                                                 
        Exit                                                       

    Timeout-->[10] sec SystemGuid-->[5A0F8F26-2BA2-11DF-9C04-001A4B07F002]
    SerialNumber-->[VM01010008          ]
{% endhighlight %}

-   Select the first partition of the disk.

{% highlight text %}
EFI Boot Maintenance Manager ver 1.10 [14.62]

Add a Boot Option.  Select a Volume

    IA64_EFI [Acpi(PNP0A03,0)/Pci(1|0)/Scsi(Pun2,Lun0)/HD(Part1,Sig7
    IA64_EFI [Acpi(PNP0A03,0)/Pci(1|0)/Scsi(Pun2,Lun0)/HD(Part3,Sig7
    Removable Media Boot [Acpi(PNP0A03,0)/Pci(1|0)/Scsi(Pun1,Lun0)]
    Load File [Acpi(PNP0A03,0)/Pci(0|0)/Mac(524FF95E0282)]         
    Load File [EFI Shell [Built-in]]                               
    Legacy Boot                                                    
    Exit
{% endhighlight %}

-   Select the first option.

{% highlight text %}
EFI Boot Maintenance Manager ver 1.10 [14.62]

Select file or change to new directory:

       03/09/10  03:45p <DIR>       4,096 EFI                      
       [Treat like Removable Media Boot]                           
    Exit
{% endhighlight %}

-   Enter the HPUX directory.

{% highlight text %}
EFI Boot Maintenance Manager ver 1.10 [14.62]

Select file or change to new directory:

       03/09/10  03:45p <DIR>       4,096 .                        
       03/09/10  03:45p <DIR>           0 ..                       
       03/09/10  03:45p <DIR>       4,096 HPUX                     
       03/09/10  03:45p <DIR>       4,096 Intel_Firmware           
       03/09/10  03:45p <DIR>       4,096 diag                     
       03/09/10  03:45p <DIR>       4,096 hp                       
       03/09/10  03:45p <DIR>       4,096 tools                    
    Exit
{% endhighlight %}

-   Select the `hpux.efi` file.

{% highlight text %}
EFI Boot Maintenance Manager ver 1.10 [14.62]

Select file or change to new directory:

       03/09/10  03:45p <DIR>       4,096 .                        
       03/09/10  03:45p <DIR>       4,096 ..                       
       03/09/10  03:45p           654,025 hpux.efi                 
       03/09/10  03:45p            24,576 nbp.efi                  
    Exit
{% endhighlight %}

-   Enter `BOOTDISK` as description and None as BootOption Data Type. Save changes.

{% highlight text %}
Filename: \EFI\HPUX\hpux.efi
DevicePath: [Acpi(PNP0A03,0)/Pci(1|0)/Scsi(Pun2,Lun0)/HD(Part1,Sig71252358-2BCD-11DF-8000-D6217B60E588)/\EFI\HPUX\hpux.efi]

IA-64 EFI Application 03/09/10  03:45p     654,025 bytes

Enter New Description:  BOOTDISK
New BootOption Data. ASCII/Unicode strings only, with max of 240 characters

Enter BootOption Data Type [A-Ascii U-Unicode N-No BootOption] :  None

Save changes to NVRAM [Y-Yes N-No]:
{% endhighlight %}

-   Go back to the EFI main menu and boot from the new option.

{% highlight text %}
EFI Boot Manager ver 1.10 [14.62] [Build: Mon Oct  1 09:27:26 2007]

Please select a boot option
HP-UX Primary Boot: 0/0/1/0.0.0
EFI Shell [Built-in]
BOOTDISK
Boot option maintenance menu

Use ^ and v to change option(s). Use Enter to select an option
Loading.: BOOTDISK
Starting: BOOTDISK

(C) Copyright 1999-2006 Hewlett-Packard Development Company, L.P.
All rights reserved

HP-UX Boot Loader for IPF  --  Revision 2.035

Press Any Key to interrupt Autoboot
\EFI\HPUX\AUTO ==> boot vmunix
Seconds left till autoboot -   0
AUTOBOOTING...> System Memory = 3066 MB
loading section 0
.................................................................................. (complete)
loading section 1
.............. (complete)
loading symbol table
loading System Directory (boot.sys) to MFS
.....
loading MFSFILES directory (bootfs) to MFS
................
Launching /stand/vmunix
SIZE: Text:41555K + Data:6964K + BSS:20747K = Total:69267K
{% endhighlight %}

-   Finally the OS will ask some questions about the network configuration and other parameters, answer what suits better
    your needing.

{% highlight text %}
_______________________________________________________________________________

                       Welcome to HP-UX!

Before using your system, you will need to answer a few questions.

The first question is whether you plan to use this system on a network.

Answer "yes" if you have connected the system to a network and are ready
to link with a network.

Answer "no" if you:

     * Plan to set up this system as a standalone (no networking).

     * Want to use the system now as a standalone and connect to a
       network later.
_______________________________________________________________________________

Are you ready to link this system to a network?

Press [y] for yes or [n] for no, then press [Enter] y
...
{% endhighlight %}

And we are done.

### Conclusions
I have to say that at the beginning the cloning system of HPVM disappointed me; but after a while I got used to it.

In my opinion the best method of the above is the second if you have one boot disk, and I really can't see a reason to have a `vg00` with several PVs on a virtual machine. If you have an IVM as template and need to produce many copies as quickly as possible this method is perfect.

Of course there is a fourth method: Our beloved Ignite-UX. But I will write about it in another post.

Juanma.
