---
title: Get the iSCSI iqn of an ESX(i) using vSphere PowerCLI
date: 2010-12-14
type: post
classes: wide
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- ESX
- ESXi
- iSCSI
- PowerCLI
- Powershell
- sysadmin
- systems administration
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

In yesterday's [post]({% post_url 2010-12-13-get-the-iscsi-iqn-of-an-esxi-using-the-cli %} "Get the iSCSI iqn of an ESX(i) using the CLI") I showed how to get the iSCSI iqn from an ESX(i) server using vSphere CLI from the vMA and from the root shell of ESX itself. Today it's turn to use PowerCLI to perform the same task.

The approach to be taken is very similar to the one we used to manage the [multipathing configuration]({% post_url 2010-12-03-managing-the-multipathing-configuration-with-vsphere-powercli %} "Managing the multipathing configuration with vSphere PowerCLI").

```
[vSphere PowerCLI] C:\> $h = Get-VMhost esx02.mlab.local
[vSphere PowerCLI] C:\> $hostview = Get-View $h.id
[vSphere PowerCLI] C:\> $storage = Get-View $hostview.ConfigManager.StorageSystem
[vSphere PowerCLI] C:\>
[vSphere PowerCLI] C:\> $storage.StorageDeviceInfo

HostBusAdapter              : {key-vim.host.ParallelScsiHba-vmhba0, key-vim.host.BlockHba-vmhba1, key-vim.host.BlockHba-vmhba32, key-vim.host.InternetScsiHba-vmhba33}
ScsiLun                     : {key-vim.host.ScsiLun-0005000000766d68626133323a303a30, key-vim.host.ScsiDisk-0000000000766d686261303a303a30}
ScsiTopology                : VMware.Vim.HostScsiTopology
MultipathInfo               : VMware.Vim.HostMultipathInfo
PlugStoreTopology           : VMware.Vim.HostPlugStoreTopology
SoftwareInternetScsiEnabled : True
DynamicType                 :
DynamicProperty             :

[vSphere PowerCLI] C:\>
[vSphere PowerCLI] C:\> $storage.StorageDeviceInfo.HostBusAdapter | select IScsiName

IScsiName                                                                                                                                                               
---------                                                                                                                                                                

iqn.1998-01.com.vmware:esx02-42b0f47e                                                                                                                                    

[vSphere PowerCLI] C:\>
```

Juanma,
