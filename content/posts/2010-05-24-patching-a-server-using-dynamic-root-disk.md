---
title: Patching a server using Dynamic Root Disk
date: 2010-05-24
categories:
- HP-UX
- Sysadmin
tags:
- DRD
- Dynamic Root Disk
- HP-UX
- LVM
- sysadmin
- systems administration
showComments: true
---

**Dynamic Root Disk**, or DRD for short, is a nice and handy tool that every HP-UX Sysadmin should know how to use. In an HPVM related post I showed how to use DRD to clone a virtual machine but today I will explain the purpose DRD was intended when it was first introduced, patching a server. I'm going to assume you have an spare disk for the task and of course have DRD installed in the server.

## Step 1 - Clone the root disk

```text
root@sheldon:/ # drd clone -x overwrite=true -v -t /dev/disk/disk2

=======  04/21/10 09:05:53 EDT  BEGIN Clone System Image (user=root)  (jobid=sheldon-01)

* Reading Current System Information
* Selecting System Image To Clone
* Selecting Target Disk
* Selecting Volume Manager For New System Image
* Analyzing For System Image Cloning
* Creating New File Systems
* Copying File Systems To New System Image
* Making New System Image Bootable
* Unmounting New System Image Clone
* System image: "sysimage_001" on disk "/dev/disk/disk2"

=======  04/21/10 09:38:48 EDT  END Clone System Image succeeded. (user=root)  (jobid=sheldon-01)

root@sheldon:/ #
```

## Step 2 - Mount the image

```text
root@sheldon:/ # drd mount

=======  04/21/10 09:41:20 EDT  BEGIN Mount Inactive System Image (user=root)  (jobid=sheldon)

 * Checking for Valid Inactive System Image
 * Locating Inactive System Image
 * Mounting Inactive System Image

=======  04/21/10 09:41:31 EDT  END Mount Inactive System Image succeeded. (user=root)  (jobid=sheldon)

root@sheldon:/ #
```

Check the mount by displaying the `drd00` volume group.

```text
root@sheldon:/ # vgdisplay drd00

VG Name                     /dev/drd00
VG Write Access             read/write     
VG Status                   available                 
Max LV                      255    
Cur LV                      8      
Open LV                     8      
Max PV                      16     
Cur PV                      1      
Act PV                      1      
Max PE per PV               4356         
VGDA                        2   
PE Size (Mbytes)            32              
Total PE                    4346    
Alloc PE                    2062    
Free PE                     2284    
Total PVG                   0        
Total Spare PVs             0              
Total Spare PVs in use      0  

root@sheldon:/ #
```

## Step 3 - Apply the patches on the mounted clone

```text
root@sheldon:/ # drd runcmd swinstall -s /tmp/patches_01.depot

=======  04/21/10 09:42:55 EDT  BEGIN Executing Command On Inactive System Image (user=root)  (jobid=sheldon)

 * Checking for Valid Inactive System Image
 * Analyzing Command To Be Run On Inactive System Image
 * Locating Inactive System Image
 * Accessing Inactive System Image for Command Execution
 * Setting Up Environment For Command Execution
 * Executing Command On Inactive System Image
 * Using unsafe patch list version 20061206
 * Starting swagentd for drd runcmd
 * Executing command: "/usr/sbin/swinstall -s /tmp/patches_01.depot"

=======  04/21/10 09:42:59 EDT  BEGIN swinstall SESSION
 (non-interactive) (jobid=sheldon-0006) (drd session)

 * Session started for user "root@sheldon".

 * Beginning Selection

 ...
 ...
 ...

=======  04/21/10 09:44:37 EDT  END swinstall SESSION (non-interactive)
 (jobid=sheldon-0006) (drd session)

 * Command "/usr/sbin/swinstall -s /tmp/patches_01.depot" completed with the return
 code "0".
 * Stopping swagentd for drd runcmd
 * Cleaning Up After Command Execution On Inactive System Image

=======  04/21/10 09:44:38 EDT  END Executing Command On Inactive System Image succeeded. (user=root)  (jobid=sheldon)

root@sheldon:/ #
```

## Step 4 - Check the installed patches on the DRD image

```text
root@sheldon:/ # drd runcmd swlist patches_01

=======  04/21/10 09:45:29 EDT  BEGIN Executing Command On Inactive System Image (user=root)  (jobid=sheldon)

 * Checking for Valid Inactive System Image
 * Analyzing Command To Be Run On Inactive System Image
 * Locating Inactive System Image
 * Accessing Inactive System Image for Command Execution
 * Setting Up Environment For Command Execution
 * Executing Command On Inactive System Image
 * Executing command: "/usr/sbin/swlist patches_01"
# Initializing...
# Contacting target "sheldon"...
#
# Target:  sheldon:/
#

 # patches_01                    1.0            ACME Patching depot
   patches_01.acme-RUN
 * Command "/usr/sbin/swlist patches_01" completed with the return code "0".
 * Cleaning Up After Command Execution On Inactive System Image

=======  04/21/10 09:45:32 EDT  END Executing Command On Inactive System Image succeeded. (user=root)  (jobid=sheldon)

root@sheldon:/ #
```

## Step 5 - Activate the image and reboot the server

At this point you only have to activate the patched image with the `drd activate` command and schedule a reboot of the server.

If you want to activate and reboot at the same time use the `-x reboot=true` option as in the example below.

```text
root@sheldon:/ # drd activate -x reboot=true

=======  04/21/10 09:52:26 EDT  BEGIN Activate Inactive System Image
 (user=root)  (jobid=sheldon)

 * Checking for Valid Inactive System Image
 * Reading Current System Information
 * Locating Inactive System Image
 * Determining Bootpath Status
 * Primary bootpath : 0/1/1/0.0.0 [/dev/disk/disk1] before activate.
 * Primary bootpath : 0/1/1/0.1.0 [/dev/disk/disk2] after activate.
 * Alternate bootpath : 0 [unknown] before activate.
 * Alternate bootpath : 0 [unknown] after activate.
 * HA Alternate bootpath : <none> [] before activate.
 * HA Alternate bootpath : <none> [] after activate.
 * Activating Inactive System Image
 * Rebooting System
```

If everything goes well after the reboot give the patched server some time, I leave this to your own criteria, before restoring the mirror.

Juanma.
