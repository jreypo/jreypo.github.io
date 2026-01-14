---
title: Get the iSCSI iqn of an ESX(i) using vSphere PowerCLI
date: 2010-12-14
tags:
- esxi
- scripting
- storage
- sysadmin
- vmware
- vsphere
showComments: true
---

In yesterday's [post]({{< ref "posts/2010-12-13-get-the-iscsi-iqn-of-an-esxi-using-the-cli.md" >}} "Get the iSCSI iqn of an ESX(i) using the CLI") I showed how to get the iSCSI iqn from an ESX(i) server using vSphere CLI from the vMA and from the root shell of ESX itself. Today it's turn to use PowerCLI to perform the same task.

The approach to be taken is very similar to the one we used to manage the [multipathing configuration]({{< ref "posts/2010-12-03-managing-the-multipathing-configuration-with-vsphere-powercli.md" >}} "Managing the multipathing configuration with vSphere PowerCLI").

```text
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
