---
title: HPVM clones first boot tasks
date: 2010-03-17
tags:
- hp-servers
- hp-ux
- storage
- sysadmin
showComments: true
---

Welcome again to **HPVM World!** my dear readers :-D

Have to say that even with the initial disappointment about `hpvmclone`, cloning IVMs was a very funny task but I believe that the after cloning tasks weren't very clear, at least for me, so I decided to write this follow up post to clarify that part.

Let's assume we already have a cloned virtual machine, in this particular case I used `dd` to clone the virtual disk and later I created the IVM and added the storage device and the other resources but it also applied to the other method with minor changes.

```text
[root@hpvmhost] ~ # hpvmstatus -P vmnode2 -d
[Virtual Machine Devices]

[Storage Interface Details]
disk:scsi:0,0,0:lv:/dev/vg_vmtest/rvmnode2disk
dvd:scsi:0,0,1:disk:/dev/rdsk/c1t4d0

[Network Interface Details]
network:lan:0,1,0xB20EBA14E76C:vswitch:localnet
network:lan:0,2,0x3E9492C9F615:vswitch:vlan02

[Misc Interface Details]
serial:com1::tty:console
[root@hpvmhost] ~ #
```

We start the virtual machine an access its console.Now we are going to follow some of the final steps of the third method described in my previous post. From the main `EFI Boot Manager` select the `Boot option maintenance menu` option.

```text
EFI Boot Manager ver 1.10 [14.62] [Build: Mon Oct  1 09:27:26 2007]

Please select a boot option

     EFI Shell [Built-in]                                           
     Boot option maintenance menu                                    

     Use ^ and v to change option(s). Use Enter to select an option
```

Select Boot from a file and the select the first partition:

```text
EFI Boot Maintenance Manager ver 1.10 [14.62]

Boot From a File.  Select a Volume

    IA64_EFI [Acpi(PNP0A03,0)/Pci(0|0)/Scsi(Pun0,Lun0)/HD(Part1,Sig
    IA64_EFI [Acpi(PNP0A03,0)/Pci(0|0)/Scsi(Pun0,Lun0)/HD(Part3,Sig7
    Removable Media Boot [Acpi(PNP0A03,0)/Pci(0|0)/Scsi(Pun1,Lun0)]
    Load File [Acpi(PNP0A03,0)/Pci(1|0)/Mac(B20EBA14E76C)]          
    Load File [Acpi(PNP0A03,0)/Pci(2|0)/Mac(3E9492C9F615)]        
    Load File [EFI Shell [Built-in]]                                
    Legacy Boot
    Exit
```

Enter the EFI directory then the HPUX directory and finally select `hpux.file`. Like I said before this part is very similar to the final steps of Method 3.

```text
EFI Boot Maintenance Manager ver 1.10 [14.62]

Select file or change to new directory:

       03/09/10  03:45p <DIR>       4,096 .                         
       03/09/10  03:45p <DIR>       4,096 ..                        
       03/10/10  04:21p           657,609 hpux.efi                  
       03/09/10  03:45p            24,576 nbp.efi                   
       Exit
```

After this the machine will boot.

```text
   Filename: \EFI\HPUX\hpux.efi
 DevicePath: [Acpi(PNP0A03,0)/Pci(0|0)/Scsi(Pun0,Lun0)/HD(Part1,Sig71252358-2BCD-11DF-8000-D6217B60E588)/\EFI\HPUX\hpux.efi]
   IA-64 EFI Application 03/10/10  04:21p     657,609 bytes

(C) Copyright 1999-2008 Hewlett-Packard Development Company, L.P.
All rights reserved

HP-UX Boot Loader for IPF  --  Revision 2.036

Press Any Key to interrupt Autoboot
\EFI\HPUX\AUTO ==> boot vmunix
Seconds left till autoboot -   0
AUTOBOOTING...> System Memory = 2042 MB
loading section 0
..................................................................................... (complete)
loading section 1
............... (complete)
loading symbol table
loading System Directory (boot.sys) to MFS
.....
loading MFSFILES directory (bootfs) to MFS
..................
Launching /stand/vmunix
SIZE: Text:43425K + Data:7551K + BSS:22118K = Total:73096K
...
```

When the VM is up login as root. The first tasks as always are to change hostname and network configuration to avoid conflicts.

Next we are going recreate `lvmtab` since the current one contains the LVM configuration of the source virtual machine. Performing a simple `vgdisplay` will show it.

```text
root@vmnode2:/# vgdisplay
vgdisplay: Warning: couldn't query physical volume "/dev/disk/disk15_p2":
The specified path does not correspond to physical volume attached to
this volume group
vgdisplay: Warning: couldn't query all of the physical volumes.
--- Volume groups 
---
VG Name                     /dev/vg00
VG Write Access             read/write     
VG Status                   available                 
Max LV                      255    
Cur LV                      8      
Open LV                     8      
Max PV                      16     
Cur PV                      1      
Act PV                      0      
Max PE per PV               3085         
VGDA                        0   
PE Size (Mbytes)            8               
Total PE                    0       
Alloc PE                    0       
Free PE                     0       
Total PVG                   0        
Total Spare PVs             0              
Total Spare PVs in use      0

root@vmnode2:/#
```

To correct this remove the `/etc/lvmtab` file and execute a new `vgscan`.

```text
root@vmnode2:/# rm /etc/lvmtab
/etc/lvmtab: ? (y/n) y
root@vmnode2:/var/tmp/software# vgscan
Creating "/etc/lvmtab".
vgscan: Couldn't access the list of physical volumes for volume group "/dev/vg00".
Physical Volume "/dev/dsk/c1t1d0" contains no LVM information

*** LVMTAB has been created successfully.
*** Do the following to resync the information on the disk.
*** #1.  vgchange -a y
*** #2.  lvlnboot -R
root@vmnode2:/#
```

Follow the recommended steps in `vgscan` output, the first step only applies if there are any other VGs in the system, if there is only `vg00` it is already active so this step is not necessary.

Running `lvnlboot -R` is mandatory since we need to recover and update the links to the logical volumes in the Boot Data Reserved Area of the booting disk.

```text
root@vmnode2:/# lvlnboot -R
Volume Group configuration for /dev/vg00 has been saved in /etc/lvmconf/vg00.conf
root@vmnode2:/#
```

Now the LVM configuration is fixed, try again the `vgdisplay` command.

```text
root@vmnode2:/# vgdisplay
--- Volume groups 
---
VG Name                     /dev/vg00
VG Write Access             read/write
VG Status                   available
Max LV                      255
Cur LV                      8
Open LV                     8
Max PV                      16
Cur PV                      1
Act PV                      1
Max PE per PV               3085
VGDA                        2
PE Size (Mbytes)            8
Total PE                    3075
Alloc PE                    2866
Free PE                     209
Total PVG                   0
Total Spare PVs             0
Total Spare PVs in use      0

root@vmnode2:/#
```

With the LVM configuration fixed the next step is to indicate the booting disk to the system.

```text
root@vmnode2:/# setboot -p /dev/disk/disk21_p2
Primary boot path set to 0/0/0/0.0x0.0x0 (/dev/disk/disk21_p2)
root@vmnode2:/#
root@vmnode2:/# setboot
Primary bootpath : 0/0/0/0.0x0.0x0 (/dev/rdisk/disk21)
HA Alternate bootpath :
Alternate bootpath :

Autoboot is ON (enabled)
root@vmnode2:/#
```

Finally reboot the virtual machine and if we did everything correctly a new boot option will be available in **EFI Boot Manager**.

```text
EFI Boot Manager ver 1.10 [14.62] [Build: Mon Oct  1 09:27:26 2007]

Please select a boot option

    HP-UX Primary Boot: 0/0/0/0.0x0.0x0                             
    EFI Shell [Built-in]                                            
    Boot option maintenance menu                                    

    Use ^ and v to change option(s). Use Enter to select an option
```

Let the system boot by itself through the new default option `HP-UX Primary Boot` and we are done.

Any feedback would be welcome.

Juanma.
