---
layout: post
title: Reconfigure an ESX(i) for HA using vSphere PowerCLI
date: 2011-02-21
type: post
published: true
status: publish
categories:
- DevOps
- Sysadmin
- Virtualization
- VMware
tags:
- API
- devops
- ESX
- ESXi
- PowerCLI
- Powershell
- sysadmin
- systems administration
- VMware
- VMware HA
- vSphere
author: juan_manuel_rey
comments: true
---

Anyone with some experience and knowledge about VMware HA knows how to perform a *Reconfigure for HA* operation in a host from the vSphere client and I'm no exception to that rule. However I never did with PowerCLI.

I created a new cluster in my homelab with a problem in one of the hosts, I fixed the problem, put my mind to work and after an hour or so digging through PowerCLI and the [vSphere API Reference Documentation](http://www.vmware.com/support/developer/vc-sdk/visdk41pubs/ApiReference/) I came up with the following easy way to do it.

First we are going to create a variable that contained the configuration of the ESXi I wanted to reconfigure.

{% highlight text %}
C:\
[vSphere PowerCLI] % $vmhost = Get-VMHost esxi06.vjlab.local
C:\
[vSphere PowerCLI] %
C:\
[vSphere PowerCLI] % $vmhost | Format-List

State                 : Connected
ConnectionState       : Connected
PowerState            : PoweredOn
VMSwapfileDatastoreId :
VMSwapfilePolicy      : Inherit
ParentId              : ClusterComputeResource-domain-c121
IsStandalone          : False
Manufacturer          : VMware, Inc.
Model                 : VMware Virtual Platform
NumCpu                : 2
CpuTotalMhz           : 5670
CpuUsageMhz           : 869
MemoryTotalMB         : 2299
MemoryUsageMB         : 868
ProcessorType         : Intel(R) Core(TM)2 Quad CPU    Q9550  @ 2.83GHz
HyperthreadingActive  : False
TimeZone              : UTC
Version               : 4.1.0
Build                 : 260247
Parent                : cluster3
VMSwapfileDatastore   :
StorageInfo           : HostStorageSystem-storageSystem-143
NetworkInfo           : esxi06:vjlab.local
DiagnosticPartition   : mpx.vmhba1:C0:T0:L0
FirewallDefaultPolicy :
ApiVersion            : 4.1
CustomFields          : {[com.hp.proliant, ]}
ExtensionData         : VMware.Vim.HostSystem
Id                    : HostSystem-host-143
Name                  : esxi06.vjlab.local
Uid                   : /VIServer=administrator@vcenter1.vjlab.local:443/VMHost=HostSystem-host-143/

C:\
[vSphere PowerCLI] %
{% endhighlight %}

Next with the cmdlet `Get-View` I retrieved the .NET objects of the host ID and stored them in another variable.

{% highlight text %}
C:\
[vSphere PowerCLI] % Get-View $vmhost.Id

Runtime             : VMware.Vim.HostRuntimeInfo
Summary             : VMware.Vim.HostListSummary
Hardware            : VMware.Vim.HostHardwareInfo
Capability          : VMware.Vim.HostCapability
ConfigManager       : VMware.Vim.HostConfigManager
Config              : VMware.Vim.HostConfigInfo
Vm                  : {}
Datastore           : {Datastore-datastore-144}
Network             : {Network-network-11}
DatastoreBrowser    : HostDatastoreBrowser-datastoreBrowser-host-143
SystemResources     : VMware.Vim.HostSystemResourceInfo
Parent              : ClusterComputeResource-domain-c121
CustomValue         : {}
OverallStatus       : red
ConfigStatus        : red
ConfigIssue         : {0}
EffectiveRole       : {-1}
Permission          : {}
Name                : esxi06.vjlab.local
DisabledMethod      : {ExitMaintenanceMode_Task, PowerUpHostFromStandBy_Task, ReconnectHost_Task}
RecentTask          : {}
DeclaredAlarmState  : {alarm-1.host-143, alarm-101.host-143, alarm-102.host-143, alarm-103.host-143...}
TriggeredAlarmState : {}
AlarmActionsEnabled : True
Tag                 : {}
Value               : {}
AvailableField      : {com.hp.proliant}
MoRef               : HostSystem-host-143
Client              : VMware.Vim.VimClient

C:\
[vSphere PowerCLI] % $esxha = Get-View $vmhost.Id
{% endhighlight %}

Now through the `$esxha` variable I invoked the method `ReconfigureHostForDAS` to reconfigure the ESXi, this method is part of the HostSystem object and its description can be found [here](http://www.vmware.com/support/developer/vc-sdk/visdk41pubs/ApiReference/vim.HostSystem.html#reconfigureDAS)in the vSphere API reference.

[![](/images/reconfigureha.png "reconfigure for HA")]({{site.url}}/images/reconfigureha.png)

As it can be seen in the above screenshot, the task is displayed in the vSphere client. You can also monitor the operation with the `Get-Task` cmdlet.

[![](/images/get-task.png "Get-Task")]({{site.url}}/images/get-task.png)

Finally I created the below script to simplify things in the future.

{% highlight powershell %}
# Reconfigure-VMHostHA.ps1
# PowerCLI script to reconfigure for VMware HA a VM Host
#
# Juan Manuel Rey - juanmanuel (dot) reyportal (at) gmail (dot) com
# http://blog.jreypo.io
#

param([string]$esx)

$vmhost = Get-VMHost $esx
$esxha = Get-View $vmhost.Id
$esxha.ReconfigureHostForDAS()
{% endhighlight %}

Juanma.
