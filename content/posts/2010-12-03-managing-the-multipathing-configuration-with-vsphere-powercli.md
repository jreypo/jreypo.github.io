---
title: Managing the multipathing configuration with vSphere PowerCLI
date: 2010-12-03
tags:
- esxi
- scripting
- storage
- sysadmin
- vmware
- vsphere
showComments: true
---

Getting the multipathing policy using PowerCLI is a very simple an straight-forward process that can be done with a few commands.

I test this procedure in the past with ESX/ESXi 3.5 and 4.0.

## Get the multipathing policy

```text
[vSphere PowerCLI] C:\> $h = get-vmhost esx01.mlab.local
[vSphere PowerCLI] C:\> $hostview = get-view $h.id
[vSphere PowerCLI] C:\> $storage = get-view $hostView.ConfigManager.StorageSystem
[vSphere PowerCLI] C:\> $storage.StorageDeviceInfo.MultipathInfo.lun | select ID,Path,Policy

Id → → Path → → → → Policy
-- → → ---- → → → → ------
vmhba0:0:0 → {vmhba0:0:0} → → → VMware.Vim.HostMultipathInfoFixedLogicalUnitPolicy
vmhba1:1:0 → {vmhba1:1:0, vmhba1:0:0} → VMware.Vim.HostMultipathInfoFixedLogicalUnitPolicy
vmhba1:0:5 → {vmhba1:1:5, vmhba1:0:5} VMware.Vim.HostMultipathInfoFixedLogicalUnitPolicy
vmhba1:0:1 → {vmhba1:1:1, vmhba1:0:1} → VMware.Vim.HostMultipathInfoFixedLogicalUnitPolicy
vmhba1:0:12 → {vmhba1:1:12, vmhba1:0:12} → VMware.Vim.HostMultipathInfoFixedLogicalUnitPolicy
```

## Change the policy from fixed to round-robin

We are going to change the policy for the LUN 12.

```text
[vSphere PowerCLI] C:\> $lunId = "vmhba1:0:12"
[vSphere PowerCLI] C:\> $storagepolicy = new-object VMware.Vim.HostMultipathInfoLogicalUnitPolicy
[vSphere PowerCLI] C:\> $storagepolicy.policy = "rr"
[vSphere PowerCLI] C:\> $storageSystem.SetMultipathLunPolicy($lunId, $policy)
```

## Finally check the new configuration

If you look closely at the last line will see that the value has change from `VMware.Vim.HostMultipathInfoFixedLogicalUnitPolicy` to `VMware.Vim.HostMultipathInfoLogicalUnitPolicy`.

```text
[vSphere PowerCLI] C:\> $h = get-vmhost "ESXIPAddress"
[vSphere PowerCLI] C:\> $hostview = get-view $h.id
[vSphere PowerCLI] C:\> $storage = get-view $hostView.ConfigManager.StorageSystem
[vSphere PowerCLI] C:\> $storage.StorageDeviceInfo.MultipathInfo.lun | select ID,Path,Policy

Id → → Path → → → → Policy
-- → → ---- → → → → ------
vmhba0:0:0 → {vmhba0:0:0} → → → VMware.Vim.HostMultipathInfoFixedLogicalUnitPolicy
vmhba1:1:0 → {vmhba1:1:0, vmhba1:0:0} → VMware.Vim.HostMultipathInfoFixedLogicalUnitPolicy
vmhba1:0:5 → {vmhba1:1:5, vmhba1:0:5} VMware.Vim.HostMultipathInfoFixedLogicalUnitPolicy
vmhba1:0:1 → {vmhba1:1:1, vmhba1:0:1} → VMware.Vim.HostMultipathInfoFixedLogicalUnitPolicy
vmhba1:0:12 → {vmhba1:1:12, vmhba1:0:12} → VMware.Vim.HostMultipathInfoLogicalUnitPolicy
```

Juanma.
