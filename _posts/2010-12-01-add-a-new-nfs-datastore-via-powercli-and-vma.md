---
layout: post
title: Add a new NFS datastore via PowerCLI and vSphere CLI from vMA
date: 2010-12-01
type: post
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- ESX
- ESX4
- ESXi
- ESXi4
- Linux
- NFS
- PowerCLI
- Powershell
- Storage
- sysadmin
- systems administration
- vMA
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

As a small follow-up to yesterday's post about [NFS shares with Openfiler]({% post_url 2010-11-30-configure-nfs-shares-in-openfiler-for-your-vsphere-homelab %} "Configure NFS shares in Openfiler for your vSphere homelab") in the following article I will show how to add a new datastore to an ESX server using vSphere CLI from the vMA appliance and PowerCLI.

### From vMA

From the vMA shell we are going to use the command `vicfg-nas`. To clarify things a bit for the newcomers, `vicfg-nas` and `esxcfg-nas` are the same command, in fact `esxcfg-nas` is no more than a link to the first.

The option to create a new datastore is -a and additionally the address/hostname of the NFS servers, the share and a label for the new datastore must be provided.

```
[vi-admin@vma ~][esx01.mlab.local]$ vicfg-nas -l
No NAS datastore found
[vi-admin@vma ~][esx01.mlab.local]$ vicfg-nas -a -o openfiler.mlab.local -s /mnt/vg_nfs/lv_nfs01/nfs_datastore1 nfs_datastore1
Connecting to NAS volume: nfs_datastore1
nfs_datastore1 created and connected.
[vi-admin@vma ~][esx01.mlab.local]$
```

When the operation is done you can check the new datastore with `vicfg-nas -l`.

```
[vi-admin@vma ~][esx01.mlab.local]$ vicfg-nas -l
nfs_datastore1 is /mnt/vg_nfs/lv_nfs01/nfs_datastore1 from openfiler.mlab.local mounted
[vi-admin@vma ~][esx01.mlab.local]$
```

### PowerCLI

In the second part of the post we are going to use **vSphere PowerCLI**, which as you already know is a PowerShell module to manage vSphere/VI3 infrastructure. I will write more about PowerCLI in the since I'm very fond with it.

The cmdlet to create the new NFS datastore is `New-Datastore` and you must provide the ESX host, the NFS server, the path of the share and a name for the datastore. Then you can check that the new datastore has been properly added with `Get-Datastore`.

[![](/images/new-datastore.png "New-DataStore")]({{site.url}}/images/new-datastore.png)

Juanma.
