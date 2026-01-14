---
title: Howto enable HP-UX 11iv3 agile naming mode in VxVM
date: 2010-06-18
tags:
- hp-ux
- storage
- sysadmin
showComments: true
---

By default Veritas Volume Manager uses HP-UX legacy naming scheme instead of the agile view one, of course for any HP-UX Sysadmin this is completely unacceptable. Below is a small procedure to change it.

Display VxVM disk information  and get the current naming scheme.

```text
root@robin:~# vxdisk list
DEVICE       TYPE            DISK         GROUP        STATUS
c0t0d0s2     auto:LVM        -            -            LVM
c0t1d0       auto:LVM        -            -            LVM
c0t2d0       auto:cdsdisk    labdg01      labdg        online
c0t3d0       auto:cdsdisk    labdg02      labdg        online
c0t4d0       auto:cdsdisk    labdg03      labdg        online
c0t5d0       auto:none       -            -            online invalid
c0t6d0       auto:none       -            -            online invalid
c0t7d0       auto:none       -            -            online invalid
c0t8d0s2     auto:hpdisk     rootdisk01   rootdg       online
c0t9d0s2     auto:hpdisk     rootdisk02   rootdg       online
root@robin:~#
root@robin:~# vxddladm get namingscheme
NAMING_SCHEME       PERSISTENCE         MODE                
===============================================
OS Native           Yes                 Legacy              
root@robin:~#
```

As you can see the mode is se tot legacy and the disks are shown with their legacy device names. To change this use again `vxddladm` command.

```text
root@robin:~# vxddladm set namingscheme=osn mode=new
```

The parameter used are `namingscheme` and `mode`. The available options for the first are:

- `ebn` - Enclosure based names.
- `osn` - Operative system names.

If `ebn` is used neither `legacy` mode nor `new` mode can be set since hardware names provided by the disk array will be used so use `osn` as naming scheme.

The second parameter is mode and of course defines which naming model will be used in the `osn` naming scheme. The following three values can be set:

- `default`
- `legacy`
- `new`

Now check the change by executing `vxdisk` and `vxddladm` commands.

```text
root@robin:~# vxdisk list
DEVICE       TYPE            DISK         GROUP        STATUS
disk4_p2     auto:LVM        -            -            LVM
disk6        auto:LVM        -            -            LVM
disk8        auto:cdsdisk    labdg01      labdg        online
disk10       auto:cdsdisk    labdg02      labdg        online
disk12       auto:cdsdisk    labdg03      labdg        online
disk14       auto:none       -            -            online invalid
disk16       auto:none       -            -            online invalid
disk18       auto:none       -            -            online invalid
disk20_p2    auto:hpdisk     rootdisk01   rootdg       online
disk22_p2    auto:hpdisk     rootdisk02   rootdg       online
root@robin:~#
root@robin:~# vxddladm get namingscheme
NAMING_SCHEME       PERSISTENCE         MODE                
===============================================
OS Native           Yes                 New                 
root@robin:~#
```

Of course the naming scheme can be set back to the legacy scheme using the same procedure.

Juanma.
