---
layout: post
title: Get the iSCSI iqn of an ESX(i) using vSphere CLI
date: 2010-12-13
type: post
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- ESX
- esxcfg-scsidevs
- ESXi
- homelab
- iSCSI
- sysadmin
- systems administration
- vicfg-iscsi
- vMA
- vmkiscsi-tool
- VMware
- vSphere
- vSphere CLI
author: juan_manuel_rey
comments: true
---

When you are trying to configure iSCSI of and ESX(i) server from the command line is clear that at some point you are going to need the iqn. Of course you can  use the vSphere Client to get the iqn but the Unix Geek inside me really wants to do it from the shell.

After a small research through the [vSphere CLI](http://www.vmware.com/support/developer/vcli/) documentation and several blogs I found this [post](http://www.2vcps.com/2009/11/27/get-iscsi-iqn-from-the-esx-command-line/) by Jon Owings ([@2vcps](http://twitter.com/2vcps)).

First list the SCSI devices available in the system to get the iSCSI hba.

```
[root@esx02 ~]# esxcfg-scsidevs -a
vmhba0  mptspi            link-n/a  pscsi.vmhba0                            (0:0:16.0) LSI Logic / Symbios Logic LSI Logic Parallel SCSI Controller
vmhba1  ata_piix          link-n/a  ide.vmhba1                              (0:0:7.1) Intel Corporation Virtual Machine Chipset
vmhba32 ata_piix          link-n/a  ide.vmhba32                             (0:0:7.1) Intel Corporation Virtual Machine Chipset
vmhba33 iscsi_vmk         online    iscsi.vmhba33                           iSCSI Software Adapter         
[root@esx02 ~]#
```

After that Jon uses the command `vmkiscsi-tool` to get the iqn.

```
[root@esx02 ~]# vmkiscsi-tool -I -l vmhba33
iSCSI Node Name: iqn.1998-01.com.vmware:esx02-42b0f47e
[root@esx02 ~]#
```

Beauty, isn't it? But I found one glitch. This method is done from the ESX root shell but how do I get the iqn from the vMA? Some of my hosts are ESXi and even for the ESX I use the vMA to perform my everyday administration tasks.

There is no `vmkiscsi-tool` command in the vMA, instead we are going to use the `vicfg-iscsi` or the `vicfg-scsidevs` command.

With `vicfg-scsidevs` we can obtain the iqn listed in the `UID` column.

```
[vi-admin@vma ~][esx02.mlab.local]$ vicfg-scsidevs -a             
Adapter_ID  Driver      UID                                     PCI      Vendor & Model
vmhba0      mptspi      pscsi.vmhba0                            (0:16.0) LSI Logic Parallel SCSI Controller
vmhba1      ata_piix    unknown.vmhba1                          (0:7.1)  Virtual Machine Chipset
vmhba32     ata_piix    ide.vmhba32                             (0:7.1)  Virtual Machine Chipset
vmhba33     iscsi_vmk   iqn.1998-01.com.vmware:esx02-42b0f47e   ()       iSCSI Software Adapter
[vi-admin@vma ~][esx02.mlab.local]$
```

And with `vicfg-iscsi` we can get the iqn providing the `vmhba` device.

```
[vi-admin@vma ~][esx02.mlab.local]$ vicfg-iscsi --iscsiname --list vmhba33
iSCSI Node Name   : iqn.1998-01.com.vmware:esx02-42b0f47e
iSCSI Node Alias  :
[vi-admin@vma ~][esx02.mlab.local]$
```

The next logical step is to use PowerCLI to retrieve the iqn, but I'll leave that for a future post.

Juanma.
