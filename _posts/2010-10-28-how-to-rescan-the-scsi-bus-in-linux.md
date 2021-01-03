---
title: How to rescan the SCSI bus in Linux
date: 2010-10-28
type: post
classes: wide
published: true
status: publish
categories:
- HP
- Linux
- Storage
- Sysadmin
tags:
- hp_rescan
- Linux
- SCSI
- Storage
- sysadmin
- systems administration
author: juan_manuel_rey
comments: true
---

You are in front of a Linux box, a VM really, with a bunch of new disks that must be configured and suddenly you remember that there is no `ioscan` in Linux, you will ask yourself, who is so stupid to create an operative system without `ioscan`?

Yes it is true, there is no `ioscan` in Linux and that means that every time you add a new disk to one of your virtual machine you have to reboot it, at least technically that is the truth. But don't worry there is a quick and dirty way to circumvent that.

From a root shell execute the following command:

```
[root@redhat ~]# echo "- - -" > /sys/class/scsi_host/<host_number>/scan
```

After that if you do a `fdsik -l` will see the new disks.

If you want to rescan your box for new fiber channel disks the command is slightly different.

```
[root@redhat ~# echo "1" > /sys/class/fc_host/host#/issue_lip
```

For the fiber channel part there are also third party utilities. HP for example provides `hp_rescan` which comes with the [Proliant Support Pack](http://h18013.www1.hp.com/products/servers/management/psp/index.html?jumpid=servers/psp).

```
[root@redhat /]# hp_rescan -h
hp_rescan: rescans LUNs on HP supported FC adapters
Usage: hp_rescan -ailh[n]

-a: rescan all adapters
-i: rescan a specific adapter instance. The specific device should be a
 SCSI host number such as "0" or "6"
-l: lists all FC adapters
-n: do not perform "scsi remove-single-device" when executing probe-luns
-h: help
[root@redhat /]#
```

If you know other ways to rescan the SCSI bus in a Linux server please comment.

Juanma.
