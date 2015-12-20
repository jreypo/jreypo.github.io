---
layout: post
title: Mirror operations with VxVM and LVM
date: 2010-05-10 16:40:50.000000000 +02:00
type: post
published: true
status: publish
categories:
- HP-UX
- Sysadmin
tags:
- HP-UX
- LVM
- Mirror-UX
- sysadmin
- systems administration
- VxVM
author: juan_manuel_rey
---

In today post I will show how to create and brake a mirrored volume in **Veritas Volume Manager** and **Logical Volume Manager**.

### LVM

Creating a mirror of a volume and later split it in LVM is quite easy an can be done with a few commands. I'm going to suppose that the original volume is already created.

-   Extend the Volume Group that contain the `lvol`

It has to be done with the same number of disks and of the same size that the ones within the VG.

{% highlight text %}
[root@sheldon] / # vgextend vg_oracle /dev/disk/disk26
Volume group "vg_oracle" has been successfully extended.
Volume Group configuration for /dev/vg_oracle has been saved in /etc/lvmconf/vg_oracle.conf
[root@sheldon] / #
{% endhighlight %}

-   Create the mirror

{% highlight text %}
[root@sheldon] / # lvextend -m 1 /dev/vg_oracle/lv_oracle /dev/disk/disk26
The newly allocated mirrors are now being synchronized. This operation will
take some time. Please wait ....
Logical volume "/dev/vg_oracle/lv_oracle" has been successfully extended.
Volume Group configuration for /dev/vg_oracle has been saved in /etc/lvmconf/vg_oracle.conf
[root@sheldon] / #
{% endhighlight %}

-   Check the configuration

{% highlight text %}
[root@sheldon] / # lvdisplay /dev/vg_oracle/lv_oracle
--- Logical volumes ---
LV Name                     /dev/vg_oracle/lv_oracle
VG Name                     /dev/vg_oracle
LV Permission               read/write                
LV Status                   available/syncd           
Mirror copies               1            
Consistency Recovery        MWC                 
Schedule                    parallel      
LV Size (Mbytes)            208             
Current LE                  13             
Allocated PE                26             
Stripes                     0       
Stripe Size (Kbytes)        0                   
Bad block                   NONE         
Allocation                  strict                    
IO Timeout (Seconds)        default             
Number of Snapshots         0  

[root@sheldon] / #
{% endhighlight %}

-   Perform the split

{% highlight text %}
[root@sheldon] / # lvsplit -s copy /dev/vg_oracle/lv_oracle
Logical volume "/dev/vg_oracle/lv_oraclecopy" has been successfully created with
character device "/dev/vg_oracle/rlv_oraclecopy".
Logical volume "/dev/vg_oracle/lv_oracle" has been successfully split.
Volume Group configuration for /dev/vg_oracle has been saved in /etc/lvmconf/vg_oracle.conf
[root@sheldon] / #
{% endhighlight %}

-   Reestablish the mirror

If the VG are 1.0 or 2.0 version the merge can not be performed if the group is in shared mode, for the 2.1 volume groups the `lvmerge` can be done in any mode.

The order to do the merge is the copy **FIRST** and the master **SECOND**. This is very important if don't want to sync the mirror in wrong direction.

{% highlight text %}
[root@sheldon] / # lvmerge /dev/vg_oracle/lv_oraclecopy /dev/vg_oracle/lv_oracle
Logical volume "/dev/vg_oracle/lv_oraclecopy" has been successfully merged
with logical volume "/dev/vg_oracle/lv_oracle".
Logical volume "/dev/vg_oracle/lv_oraclecopy" has been successfully removed.
Volume Group configuration for /dev/vg_oracle has been saved in /etc/lvmconf/vg_oracle.conf
[root@sheldon] / #
{% endhighlight %}

### VXVM

The process in VxVM is in many ways similar to the LVM one.

-   Add a new disk/disks to the `diskgroup`

Launch `vxdiskadm` tool and select `Add or initialize one or more disks`.

{% highlight text %}
[root@sheldon] / # vxdiskadm

Volume Manager Support Operations
Menu: VolumeManager/Disk

 1      Add or initialize one or more disks
 2      Remove a disk
 3      Remove a disk for replacement
 4      Replace a failed or removed disk
 5      Mirror volumes on a disk
 6      Move volumes from a disk
 7      Enable access to (import) a disk group
 8      Remove access to (deport) a disk group
 9      Enable (online) a disk device
 10     Disable (offline) a disk device
 11     Mark a disk as a spare for a disk group
 12     Turn off the spare flag on a disk
 13     Remove (deport) and destroy a disk group
 14     Unrelocate subdisks back to a disk
 15     Exclude a disk from hot-relocation use
 16     Make a disk available for hot-relocation use
 17     Prevent multipathing/Suppress devices from VxVM's view
 18     Allow multipathing/Unsuppress devices from VxVM's view
 19     List currently suppressed/non-multipathed devices
 20     Change the disk naming scheme
 21     Change/Display the default disk layouts
 22     Mark a disk as allocator-reserved for a disk group
 23     Turn off the allocator-reserved flag on a disk
 list   List disk information

 ?      Display help about menu
 ??     Display help about the menuing system
 q      Exit from menus

Select an operation to perform:  1
{% endhighlight %}

Enter the disk and answer the questions according to your configuration and exit the tool when the process is done.

{% highlight text %}
Add or initialize disks
Menu: VolumeManager/Disk/AddDisks

 Use this operation to add one or more disks to a disk group.  You can
 add the selected disks to an existing disk group or to a new disk group
 that will be created as a part of the operation. The selected disks may
 also be added to a disk group as spares. Or they may be added as
 nohotuses to be excluded from hot-relocation use. The selected
 disks may also be initialized without adding them to a disk group
 leaving the disks available for use as replacement disks.

 More than one disk or pattern may be entered at the prompt.  Here are
 some disk selection examples:

 all:          all disks
 c3 c4t2:      all disks on both controller 3 and controller 4, target 2
 c3t4d2:       a single disk (in the c#t#d# naming scheme)
 xyz_0:        a single disk (in the enclosure based naming scheme)
 xyz_:         all disks on the enclosure whose name is xyz

 disk#:        a single disk (in the new naming scheme)

Select disk devices to add: [<pattern-list>,all,list,q,?]  disk28
Here is the disk selected.  Output format: [Device_Name]

 disk28

Continue operation? [y,n,q,?] (default: y)  

 You can choose to add this disk to an existing disk group, a
 new disk group, or leave the disk available for use by future
 add or replacement operations.  To create a new disk group,
 select a disk group name that does not yet exist.  To leave
 the disk available for future use, specify a disk group name
 of "none".

Which disk group [<group>,none,list,q,?] (default: none)  dg_sap

Use a default disk name for the disk? [y,n,q,?] (default: y)  n

Add disk as a spare disk for dg_sap? [y,n,q,?] (default: n)  

Exclude disk from hot-relocation use? [y,n,q,?] (default: n)  

Add site tag to disk? [y,n,q,?] (default: n)  

 The selected disks will be added to the disk group dg_sap with
 disk names that you will specify interactively.

 disk28

Continue with operation? [y,n,q,?] (default: y)  

 Initializing device disk28.

Enter desired private region length
[<privlen>,q,?] (default: 32768)  

Enter disk name for disk28 [<name>,q,?] (default: dg_sap02)  

 VxVM  NOTICE V-5-2-88
Adding disk device disk28 to disk group dg_sap with disk
 name dg_sap02.

Add or initialize other disks? [y,n,q,?] (default: n)
{% endhighlight %}

-   Check the configuration.

{% highlight text %}
[root@sheldon] / # vxprint -g dg_sap
TY NAME         ASSOC        KSTATE   LENGTH   PLOFFS   STATE    TUTIL0  PUTIL0
dg dg_sap       dg_sap       -        -        -        -        -       -

dm dg_sap01     disk27       -        228224   -        -        -       -
dm dg_sap02     disk28       -        228224   -        -        -       -

v  sapvol       fsgen        ENABLED  204800   -        ACTIVE   -       -
pl sapvol-01    sapvol       ENABLED  204800   -        ACTIVE   -       -
sd dg_sap01-01  sapvol-01    ENABLED  204800   0        -        -       -
[root@sheldon] / #
{% endhighlight %}

-   Create the mirror.

{% highlight text %}
[root@sheldon] / # vxassist -g dg_sap mirror sapvol dg_sap02
[root@sheldon] / #
[root@sheldon] / # vxprint -g dg_sap                        
TY NAME         ASSOC        KSTATE   LENGTH   PLOFFS   STATE    TUTIL0  PUTIL0
dg dg_sap       dg_sap       -        -        -        -        -       -

dm dg_sap01     disk27       -        228224   -        -        -       -
dm dg_sap02     disk28       -        228224   -        -        -       -

v  sapvol       fsgen        ENABLED  204800   -        ACTIVE   -       -
pl sapvol-01    sapvol       ENABLED  204800   -        ACTIVE   -       -
sd dg_sap01-01  sapvol-01    ENABLED  204800   0        -        -       -
pl sapvol-02    sapvol       ENABLED  204800   -        ACTIVE   -       -
sd dg_sap02-01  sapvol-02    ENABLED  204800   0        -        -       -
[root@sheldon] / #
{% endhighlight %}

-   Break the mirror.

To do this just disassociate the corresponding `plex` from the volume.

{% highlight text %}
[root@sheldon] / # vxplex -g dg_sap dis sapvol-02
[root@sheldon] / # vxprint -g dg_sap             
TY NAME         ASSOC        KSTATE   LENGTH   PLOFFS   STATE    TUTIL0  PUTIL0
dg dg_sap       dg_sap       -        -        -        -        -       -

dm dg_sap01     disk27       -        228224   -        -        -       -
dm dg_sap02     disk28       -        228224   -        -        -       -

pl sapvol-02    -            DISABLED 204800   -        -        -       -
sd dg_sap02-01  sapvol-02    ENABLED  204800   0        -        -       -

v  sapvol       fsgen        ENABLED  204800   -        ACTIVE   -       -
pl sapvol-01    sapvol       ENABLED  204800   -        ACTIVE   -       -
sd dg_sap01-01  sapvol-01    ENABLED  204800   0        -        -       -
[root@sheldon] / #
{% endhighlight %}

-   Reattach the `plex` to the volume and reestablish the mirror.

{% highlight text %}
[root@sheldon] / # vxplex -g dg_sap att sapvol sapvol-02
{% endhighlight %}

And we are done for now, more VxVM stuff in a future post.

Juanma.
