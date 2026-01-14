---
title: Remove an inactive NFS datastore using vSphere CLI from vMA
date: 2010-12-01
tags:
- esxi
- homelab
- linux
- storage
- sysadmin
- vmware
- vsphere
showComments: true
---

This post is mostly for self-reference but may be someone would find it useful. Last night I decided to change the IP address of one of the Openfiler instances in my homelab and instead of previously removing the NFS shares from the ESX servers I simply made the changes.

After a restart of the network services in the Openfiler server to commit the changes I found that the ESX servers saw the datastore as inactive.

First I tried to remove it from the vSphere Client and I received the following error message:

[![](/images/inactive_datastore.png "Inactive datastore")](/images/inactive_datastore.png)

I quickly switched to an SSH session in the vMA to check the state of the datastore, it appeared as `not mounted`.

```text
[vi-admin@vma /][esx01.mlab.local]$ esxcfg-nas -l
nfs_datastore1 is /mnt/vg_nfs/lv_nfs01/nfs_datastore1 from openfiler.mlab.local not mounted
[vi-admin@vma /][esx01.mlab.local]$
```

At this point I used `esxcfg-nas` command to remove the datastore.

```text
[vi-admin@vma /][esx01.mlab.local]$ esxcfg-nas -d nfs_datastore1
NAS volume nfs_datastore1 deleted.
[vi-admin@vma /][esx01.mlab.local]$ esxcfg-nas -l
No NAS datastore found
[vi-admin@vma /][esx01.mlab.local]$
```

Very easy, isn't it? Oh by the way this just confirm one of my personal beliefs *Where there is shell, there is a way* ;-)

Juanma.
