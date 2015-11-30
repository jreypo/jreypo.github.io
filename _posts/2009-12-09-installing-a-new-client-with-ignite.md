---
layout: post
title: Installing a new client with Ignite
date: 2009-12-09 13:50:46.000000000 +01:00
type: post
published: true
status: publish
categories:
- HP-UX
tags:
- HP-UX
- Ignite
- nPartitions
author: juan_manuel_rey
---

Along my career as HP-UX administrator one of the most useful tools has been **Ignite-UX**. This is, IMHO, the most powerful backup/recovery/deployment tool above any other currently present in the Unix OS family (Solaris JumpStart, RH Kickstart, AIX NIM...).

It allows you to deploy several clients simultaneously, create install images (golden images), perform OS backups for disaster recovery, etc. In this post I will show how to manually set-up a new Itanium server with Ignite-UX from the "ignited image" of another server. The example shows an Itanium partionable system, the procedure for a non-partionable client is slightly different and I will talk about it in a future post.

In the client:

-   Boot your new server into the EFI Shell and with the `lanaddress` command search for our MAC:

{% highlight text %}
Shell> lanaddress

LAN Address Information

 LAN Address        Path
 -----------------  ----------------------------------------
 *Mac(XXXXXXXXXXXX)  Acpi(HWP0002,PNP0A03,100)/Pci(1|0)/Mac(XXXXXXXXXXXX)
 Mac(YYYYYYYYYYYY)  Acpi(HWP0002,PNP0A03,100)/Pci(1|1)/Mac(YYYYYYYYYYYY)
 Mac(000000000000)  Acpi(HWP0002,PNP0A03,200)/Pci(2|0)/Mac(000000000000)
 Mac(00AA00AA00AA)  Acpi(HWP0002,PNP0A03,200)/Pci(2|1)/Mac(00AA00AA00AA)
{% endhighlight %}

-   Create a new Direct Boot Profile:

{% highlight text %}
Shell> dbprofile -dn newserver -sip 10.10.10.2 -cip 10.10.10.35 -gip 10.31.4.1 -m 255.255.255.0 -b "/opt/ignite/boot/nbp.efi"
Creating profile newserver
{% endhighlight %}

The `dbprofile` command is exclusive for partionable servers:

1.  `-dn`  Name of the new profile
2.  `-sip` IP address of the Ignite-UX server.
3.  `-cip` Address of the client.
4.  `-gip` Gateway.
5.  `-m` Network Mask.
6.  `-b` Boot file name.

In the Ignite server:

-   Create the directory `/var/opt/ignite/clients/<0xMAC_of_the_client>`.

{% highlight text %}
[ignite]/var/opt/ignite/clients # mkdir 0xXXXXXXXXXXXX
{% endhighlight %}

-   Put bin:sys as owner:group of the new directory.

{% highlight text%}
[ignite]/var/opt/ignite/clients # chown bin:sys 0xXXXXXXXXXXXX
{% endhighlight %}

-   Create a link `<client> -> <0xMAC_of_the_client>` in the same location.

{% highlight text%}
[ignite]/var/opt/ignite/clients # ln -s 0xXXXXXXXXXXXX newserver
{% endhighlight %}

-   Set `bin:bin` as owner of the link.

{% highlight text%}
[ignite]/var/opt/ignite/clients # chown -h bin:bin newserver
{% endhighlight %}

-   Copy the data from the "source client" to the "target client".

{% highlight text%}
[ignite]/var/opt/ignite/clients/source_server # find CINDEX recovery | cpio -pdvma ../newserver
../newserver/CINDEX
../newserver/recovery/client_status
../newserver/recovery/2009-03-20,10:41/recovery.log
../newserver/recovery/2009-03-20,10:41/archive_content
../newserver/recovery/2009-03-20,10:41/system_cfg
../newserver/recovery/2009-03-20,10:41/control_cfg
../newserver/recovery/2009-03-20,10:41/flist
../newserver/recovery/2009-03-20,10:41/archive_cfg
../newserver/recovery/2009-03-20,10:41/manifest
../newserver/recovery/defaults
../newserver/recovery/archive_content
../newserver/recovery/2009-04-07,10:50/recovery.log
../newserver/recovery/2009-04-07,10:50/archive_content
../newserver/recovery/2009-04-07,10:50/system_cfg
../newserver/recovery/2009-04-07,10:50/control_cfg
../newserver/recovery/2009-04-07,10:50/flist
../newserver/recovery/2009-04-07,10:50/archive_cfg
../newserver/recovery/2009-04-07,10:50/manifest
../newserver/recovery/2009-04-07,11:50/recovery.log
../newserver/recovery/2009-04-07,11:50/archive_content
../newserver/recovery/2009-04-07,11:50/system_cfg
../newserver/recovery/2009-04-07,11:50/control_cfg
../newserver/recovery/2009-04-07,11:50/flist
../newserver/recovery/2009-04-07,11:50/archive_cfg
../newserver/recovery/2009-04-07,11:50/manifest
65202 blocks
{% endhighlight %}

Now we have to share the new directory via NFS. In HP-UX 11.31 is quite simple, add the corresponding line in `/etc/dfs/dfstab` and execute the `shareall -F nfs` command.

In our example server it will shows like this:

{% highlight text%}
[ignite]/etc/dfs # cat dfstab
#       place share(1M) commands here for automatic execution #       on entering init state 3.
#
#       share [-F fstype] [ -o options] [-d "<text>"] <pathname>
#       .e.g,
#       share  -F nfs  -o rw=engineering  -d "home dirs"  /home
share -F nfs -o anon=2 /var/opt/ignite/clients
share -F nfs -o sec=sys,anon=2,rw=newserver.my.dom /var/opt/ignite/recovery/archives/newserver
{% endhighlight %}

If the newserver hostname is not included in your DNS you have to add it to the `/etc/hosts` of the Ignite server.

The next step is in the client EFI Shell, we boot it with `lanboot` command.

{% highlight text%}
Shell> lanboot select -dn newserver
 01 Acpi(HWP0002,PNP0A03,100)/Pci(1|0)/Mac(XXXXXXXXXXXX)
02 Acpi(HWP0002,PNP0A03,100)/Pci(1|1)/Mac(YYYYYYYYYYYY)
03 Acpi(HWP0002,PNP0A03,200)/Pci(2|0)/Mac(000000000000)
04 Acpi(HWP0002,PNP0A03,200)/Pci(2|1)/Mac(00AA00AA00AA)
Select Desired LAN: 01
Selected Acpi(HWP0002,PNP0A03,100)/Pci(1|0)/Mac(XXXXXXXXXXXX)

Client MAC Address: XXXXXXXXXXXX
Client IP Address: 10.10.10.35
Subnet Mask: 255.255.255.0
BOOTP Server IP Address: 10.10.10.2
DHCP Server IP Address: 0.0.0.0
Boot file name: /opt/ignite/boot/nbp.efi

Retrieving File Size.
Retrieving File (TFTP).
@(#) HP-UX IA64 Network Bootstrap Program Revision 1.1
Downloading HPUX bootloader
Starting HPUX bootloader
Obtaining size of fpswa.efi   (328192 bytes)
Downloading file  fpswa.efi   (328192 bytes)

(C) Copyright 1999-2008 Hewlett-Packard Development Company, L.P.
All rights reserved

HP-UX Boot Loader for IPF  --  Revision 2.037

Booting from Lan
Obtaining size of AUTO   (226 bytes)
Downloading file  AUTO   (226 bytes)
Obtaining size of AUTO   (226 bytes)
Downloading file  AUTO   (226 bytes)

Obtaining size of AUTO   (226 bytes)
Downloading file  AUTO   (226 bytes)
 1.  target OS is B.11.23 IA
 2.  target OS is B.11.31 IA
 3.  Exit Boot Loader

Choose an operating system to install that your hardware supports:2
Obtaining size of AUTO   (226 bytes)
Downloading file  AUTO   (226 bytes)
Obtaining size of Rel_B.11.31/IINSTALL   (51685533 bytes)
Downloading file  Rel_B.11.31/IINSTALL   (51685533 bytes)
> System Memory = 12257 MB
loading section 0
.................................................................................................... (complete)
loading section 1
...................... (complete)
loading symbol table
Obtaining size of Rel_B.11.31/IINSTALLFS   (61341696 bytes)
Downloading file  Rel_B.11.31/IINSTALLFS   (61341696 bytes)
loading ram disk file (Rel_B.11.31/IINSTALLFS).
.....................................................................................................................
 (complete)

================================================================================
WARNING: Multiple console output devices are configured. If this message
remains on the screen for more than a few minutes, then this is not the
device in use by HP-UX as the console output device. If you would like this
device to be the one used by HP-UX as the console output device, reboot and
use the EFI boot manager or the EFI 'conconfig' command to select this device
and deconfigure the others.
================================================================================

Launching Rel_B.11.31/IINSTALL
SIZE: Text:50974K + Data:11077K + BSS:25419K = Total:87471K
Console is on Serial Device - via PCDP
Booting kernel...

krs_read_mfs: Error 5 opening MFS.
Loaded ACPI revision 2.0 tables.
krs_read_mfs: Error 5 opening MFS.

Memory Class Setup
-------------------------------------------------------------------------
Class     Physmem              Lockmem              Swapmem
-------------------------------------------------------------------------
System :  11659 MB             11659 MB             11659 MB
Kernel :  11659 MB             11659 MB             11659 MB
User   :  10803 MB             9577 MB              9615 MB
-------------------------------------------------------------------------

ktracer is off until requested.
Installing Socket Protocol families AF_INET and AF_INET6
Kernel EVM initialized
sec_init(): kernel RPC authentication/security initialization.
secgss_init():  kernel RPCSEC_GSS security initialization.
rpc_init(): kernel RPC initialization.
rpcmod_install(): kernel RPC STREAMS module "rpcmod" installation. ...(driver_install)
NOTICE: nfs_client_pv3_install(): nfs3 File system was registered at index 10.
NOTICE: nfs_client_pv4_install(): nfs4 File system was registered at index 11.

 System Console is on the Built-In Serial Interface
igelan2: INITIALIZING HP PCI-X 1000Mbps Dual-port Built-in at hardware path 0/2/2/0
igelan0: INITIALIZING HP PCI-X 1000Mbps Dual-port Built-in at hardware path 0/1/1/0
igelan1: INITIALIZING HP PCI-X 1000Mbps Dual-port Built-in at hardware path 0/1/1/1
igelan3: INITIALIZING HP PCI-X 1000Mbps Dual-port Built-in at hardware path 0/2/2/1
AF_INET socket/streams output daemon running, pid 35
afinet_prelink: module installed
Starting the STREAMS daemons-phase 1
 Swap device table:  (start & size given in 512-byte blocks)
 entry 0 - auto-configured on root device; ignored - no room
WARNING: No swap device configured, so dump cannot be defaulted to primary swap.
WARNING: No dump devices are configured.  Dump is disabled.
Create STCP device files
Starting the STREAMS daemons-phase 2
 $Revision: vmunix:    B.11.31_LR FLAVOR=perf nfsauth: lookupname: 2

Memory Information:
 physical page size = 4096 bytes, logical page size = 4096 bytes
 Physical: 12551908 Kbytes, lockable: 9810760 Kbytes, available: 10807748 Kbytes

 * Preparing to execute init...
=======  04/07/09 06:33:05 EDT  HP-UX Installation Initialization.
 @(#)Ignite-UX Revision C.7.8.201
 @(#)ignite/launch (opt) Revision:
 /branches/IUX_RA0903/ignite/src@76987 Last Modified: 2009-02-05
 15:45:55 -0700 (Thu, 05 Feb 2009)
 * Configuring RAM filesystems...
 * No SAS disk/LUN swaps required, already in physical location order.
 * Scanning system for IO devices...
 * Boot device is: 0/1/1/0
NOTE:    Primary path not currently set to an existing disk device.
 * Setting keyboard language.

A USB interface has been detected on this system.
In order to use a keyboard on this interface, you must specify
a language mapping which will be used by X windows and
the Internal Terminal Emulator (ITE).
The characters "1234567890" will appear as "!@#$^&*()"
on keyboards that use the shift key to type a number.
Your choice will be stored in the file /etc/kbdlang

 1) USB_PS2_DIN_Belgian                  2) USB_PS2_DIN_Belgian_Euro
 3) USB_PS2_DIN_Danish                   4) USB_PS2_DIN_Danish_Euro
 5) USB_PS2_DIN_Euro_Spanish             6) USB_PS2_DIN_Euro_Spanish_Euro
 7) USB_PS2_DIN_French                   8) USB_PS2_DIN_French_Euro
 9) USB_PS2_DIN_German                  10) USB_PS2_DIN_German_Euro
11) USB_PS2_DIN_Italian                 12) USB_PS2_DIN_Italian_Euro
13) USB_PS2_DIN_JIS_109                 14) USB_PS2_DIN_Korean
15) USB_PS2_DIN_Norwegian               16) USB_PS2_DIN_Norwegian_Euro
17) USB_PS2_DIN_S_Chinese               18) USB_PS2_DIN_Swedish
19) USB_PS2_DIN_Swedish_Euro            20) USB_PS2_DIN_Swiss_French2_Euro
21) USB_PS2_DIN_Swiss_German2           22) USB_PS2_DIN_Swiss_German2_Euro
23) USB_PS2_DIN_T_Chinese               24) USB_PS2_DIN_UK_English
25) USB_PS2_DIN_UK_English_Euro         26) USB_PS2_DIN_US_English
27) USB_PS2_DIN_US_English_Euro

Enter the number of the language you want:6

You have selected the keyboard language USB_PS2_DIN_Euro_Spanish_Euro.
Please confirm your choice by pressing RETURN or enter a new number:
---------------------------------------------------------------------------------

 Welcome to Ignite-UX!

 Use the <tab> key to navigate between fields, and the arrow keys
 within fields.  Use the <return/enter> key to select an item.
 Use the <return/enter> or <space-bar> to pop-up a choices list.  If the
 menus are not clear, select the "Help" item for more information.

 Hardware Summary:         System Model: ia64 hp BL860c
 +----------------------+---------------+--------------------+[ Scan Again  ]
 | Disks: 1  (  136.0GB)| Floppies: 0   | LAN cards:   4     |
 | CD/DVDs:        0    | Tapes:    0   | Memory:    12257Mb |
 | Graphics Ports: 1    | IO Buses: 5   | CPUs:        4     |[ H/W Details ]
 +----------------------+---------------+--------------------+
                   [      Install HP-UX       ]

               [   Run an Expert Recovery Shell   ]

                   [    Advanced Options      ]

      [  Reboot  ]                              [  Help  ]
{% endhighlight %}

Now select `"Install HP-UX"` option. And the following screen appears where we select the `"OK"` option:

{% highlight text%}
User Interface and Media Options

This screen lets you pick from options that will determine if an
Ignite-UX server is used, and your user interface preference.

User Interface Options:
[   ]  Guided Installation   (very basic installs - deprecated mode)
[ * ]  Advanced Installation (recommended for disk and filesystem management)
[   ]  No user interface - setup basic networking, use defaults and go
[   ]  Remote graphical interface running on the Ignite-UX server

Hint: If you need to make LVM size changes, or want to set the
final networking parameters during the install, you will
need to use the Advanced mode (or remote graphical interface).

[   OK   ]                  [ Cancel ]                         [  Help  ]
{% endhighlight %}

In the next screen we select the corresponding lan interface:

    LAN Interface Selection

{% highlight text%}
 More than one network interface was detected on the system.  You
 will need to select the interface to enable.  Only one interface
 can be enabled, and it must be the one connected to the network
 that can be used in contacting the install and/or SD servers.

 Use the <tab> and/or arrow keys to move to the desired LAN device
 to enable, then press <Return>.

 HW Path    Interface   Station Address  Description
 ----------------------------------------------------------

 [ 0/1/1/0     lan0     0x001E0BFCEE94   HP_PCI-X_1000Mbps_Dual-port_Bu ]

 [ 0/1/1/1     lan1     0x001E0BFCEE95   HP_PCI-X_1000Mbps_Dual-port_Bu ]

 [ 0/2/2/0     lan2     0x001E0BFCEE92   HP_PCI-X_1000Mbps_Dual-port_Bu ]

 [ 0/2/2/1     lan3     0x001E0BFCEE93   HP_PCI-X_1000Mbps_Dual-port_Bu ]
{% endhighlight %}

It starts to search for the DHCP server, press Crtl-C to stop it. The install process prompts us for the target client IP and hostname.

{% highlight text%}
* Could not get DHCP information.  No host specific network defaults
 will be supplied.  (dhcpclient returned: 5)

--------------------------------------------------------------------------------------------
 NETWORK CONFIGURATION

 This system's hostname: nfsux02

 Internet protocol address (eg. 15.2.56.1) of this host: 10.10.10.35

 Default gateway routing internet protocol address: 10.10.10.1

 The subnet mask (eg. 255.255.248.0 or 0xfffff800): 255.255.255.0

 IP address of the Ignite-UX server system: 10.10.10.2

 Is this networking information only temporary?  [ No  ]

 [   OK   ]                  [ Cancel ]                         [  Help  ]
{% endhighlight %}

The new client is added to the Ignite-UX server. It shows a warning screen informing that the disk device is not present in the system and it will substituted, this is normal since we are installing from an Ignite image of other server. Select `"OK"`.

{% highlight text%}
----------------------------------------------------------------------------------------------
+                           /opt/ignite/bin/itool ()                           +
¦                                                                              ¦
¦ +-------++----------++--------++-------------++----------+                   ¦
¦ ¦ Basic ¦¦ Software ¦¦ System ¦¦ File System ¦¦ Advanced ¦                   ¦
¦ ¦+                                 Note                                  +--+¦
¦ ¦¦                                                                       ¦  ¦¦
¦ ¦¦ Message From: /opt/ignite/bin/itool ()                                ¦  ¦¦
¦ ¦¦                                                                       ¦  ¦¦
¦ ¦¦ NOTE: The disk with Device Specifier:                                 ¦  ¦¦
¦ ¦¦ "WWID='0x600508e00000000009e9a4d0f3569501'                            ¦  ¦¦
¦ ¦¦ PHYS_LOC='SAS:VOL019556F3D0A4E909:ENC01:BAYS01,02'                    ¦  ¦¦
¦ ¦¦ HW_PATH='0/2/1/0.0x19556f3d0a4e909.0x0'" does not exist on the system ¦  ¦¦
¦ ¦¦ and is being substituted by the disk at:                              ¦  ¦¦
¦ ¦¦ "WWID='0x600508e000000000a99ff86eef54f309'                            ¦  ¦¦
¦ ¦¦ PHYS_LOC='SAS:VOL09F354EF6EF89FA9:ENC01:BAYS01,02'                    ¦  ¦¦
¦ ¦¦ HW_PATH='0/2/1/0.0x9f354ef6ef89fa9.0x0'" (HP_IR_Volume)               ¦  ¦¦
¦ ¦¦                                                                       ¦ ]¦¦
¦ ¦¦-----------------------------------------------------------------------¦  ¦¦
¦ +¦                              [[ OK    ]]                              ¦--+¦
¦  +-----------------------------------------------------------------------+   ¦
¦------------------------------------------------------------------------------¦
¦ [  Go!   ]                       [ Cancel ]                       [  Help  ] ¦
+------------------------------------------------------------------------------+

------------------------------------------------------------------------------------------------
{% endhighlight %}

In the system tab enter the new hostname and IP address.

{% highlight text%}
------------------------------------------------------------------------------------------------
+                           /opt/ignite/bin/itool ()                           +
¦                                                                              ¦
¦ +-------++----------++--------++-------------++----------+                   ¦
¦ ¦ Basic ¦¦ Software ¦¦ System ¦¦ File System ¦¦ Advanced ¦                   ¦
¦ +--------------------/        \---------------------------------------------+¦
¦ ¦                                                                           ¦¦
¦ ¦  Final System Parameters:  [ Set parameters now   ->]                     ¦¦
¦ ¦                                                                           ¦¦
¦ ¦  +------------------------------------------------------------------+     ¦¦
¦ ¦  ¦  Hostname:  nfsux02                                              ¦     ¦¦
¦ ¦  ¦                                                                  ¦     ¦¦
¦ ¦  ¦IP Address:  10.31.4.70      Subnet Mask:  0xffffff00             ¦     ¦¦
¦ ¦  ¦                                                                  ¦     ¦¦
¦ ¦  ¦      Time:  12:42  Day:  07  Month:  [ April     ->] Year:  2009 ¦     ¦¦
¦ ¦  +------------------------------------------------------------------+     ¦¦
¦ ¦    [ Set Time Zone (MET-1METDST ]   [     Network Services...    ]        ¦¦
¦ ¦    [    Set Root Password...    ]   [ Additional Interface(s)... ]        ¦¦
¦ ¦                                                                           ¦¦
¦ +---------------------------------------------------------------------------+¦
¦      [ Show Summary...  ]                          [ Reset Configuration ]   ¦
¦------------------------------------------------------------------------------¦
¦ [  Go!   ]                       [ Cancel ]                       [  Help  ] ¦
+------------------------------------------------------------------------------+
------------------------------------------------------------------------------------------------
{% endhighlight %}

Now review the other parameters such swap space, filesystems, root password, etc. If everything is fine select `"Go!"`, it will ask for confirmation:

{% highlight text%}
------------------------------------------------------------------------------------------------
++                             itool Confirmation                             ++
¦¦                                                                            ¦¦
¦¦ All data will be destroyed on the following disks:                         ¦¦
¦¦                                                                            ¦¦
¦¦   Addr                                             Disk Size(M             ¦¦
¦¦ +--------------------------------------------------------------+           ¦¦
¦¦ ¦ 0/2/1/0:SAS:VOL09F354EF6EF89FA9:ENC01:BAYS01,02  139236 MB   ^           ¦¦
¦¦ ¦                                                                          ¦¦
¦¦ ¦                                                              v           ¦¦
¦¦ +<                                                            >+           ¦¦
¦¦                                                                            ¦¦
¦¦ The results of the preinstall analysis are:                                ¦¦
¦¦                                                                            ¦¦
¦¦ +--------------------------------------------------------------+           ¦¦
¦¦ ¦ NOTE: Free space (10485760KB) in "/var/adm/crash" where      ^           ¦¦
¦¦ ¦ /var/adm/crash is located is less than system memory                     ¦¦
¦¦ ¦ (12551908KB). This should be enough space to capture at                  ¦¦
¦¦ ¦ least a single dump (and likely more than that if the dump               ¦¦
¦¦ ¦ is selective and/or compressed) in the event of a system                 ¦¦
¦¦ ¦ crash. Additional space may be required to uncompress the    v           ¦¦
¦¦ +--------------------------------------------------------------+           ¦¦
¦¦----------------------------------------------------------------------------¦¦
+¦ [  Go!   ]                      [ < Back ]                      [  Help  ] ¦+
 +----------------------------------------------------------------------------+
------------------------------------------------------------------------------------------------
{% endhighlight %}

Select `"Go!"` again and the installation will begin. The warning message about `/var/adm/crash` filesystem is completely normal, the creation of that filesystem is performed after the installation of the Operative System, at least I use to.

After a while if everything goes as expected you should have a new HP-UX server installed and ready, now it is time to review all the configuration parameters to check that nothing remains of the source client.

Hope you find this post useful, in the future I will write about the `vg00` mirroring and other post-install tasks.

Juanma.
