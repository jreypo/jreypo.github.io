---
title: Managing your vSphere cluster configuration with PowerCLI
date: 2011-06-06
type: post
classes: wide
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- PowerCLI
- Powershell
- sysadmin
- systems administration
- VMware
- VMware DRS
- VMware HA
- vSphere
author: juan_manuel_rey
comments: true
---

Managing VMware vSphere cluster configuration with the vSphere Client can be, sometimes, a tedious task. In this post I'll show you how to use PowerCLI to manage your cluster's configuration. I did not pretend to write a full detailed guide but just to show the most common tasks I perform at work.

The first thing  to do is to properly setup the basics that will allow us to interact with the cluster. First get your cluster basic configuration and store it in a variable, `$cldl380` in the example.

```powershell-interactive
[vSphere PowerCLI] C:\> $cldl380 = get-cluster cluster-dl380-01
```

Use the cmdlet to `Get-View` to get the .Net objects of the cluster and store the result in another variable.

```powershell-interactive
[vSphere PowerCLI] C:\> $viewdl380 = get-view $cldl380.Id
[vSphere PowerCLI] C:\> $viewdl380

Configuration       : VMware.Vim.ClusterConfigInfo
Recommendation      : {}
DrsRecommendation   : {}
MigrationHistory    : {}
ActionHistory       : {}
DrsFault            : {}
ResourcePool        : ResourcePool-resgroup-36
Host                : {HostSystem-host-37, HostSystem-host-39}
Datastore           : {Datastore-datastore-11, Datastore-datastore-12, Datastore-datastore-13, Datastore-datastore-38...}
Network             : {Network-network-21, Network-network-60, Network-network-22, Network-network-16}
Summary             : VMware.Vim.ClusterComputeResourceSummary
EnvironmentBrowser  : EnvironmentBrowser-envbrowser-35
ConfigurationEx     : VMware.Vim.ClusterConfigInfoEx
Parent              : Folder-group-h4
CustomValue         : {}
OverallStatus       : red
ConfigStatus        : red
ConfigIssue         : {0, 0}
EffectiveRole       : {-1}
Permission          : {}
Name                : cluster-dl380-01
DisabledMethod      : {}
RecentTask          : {}
DeclaredAlarmState  : {alarm-1.domain-c35, alarm-11.domain-c35, alarm-12.domain-c35, alarm-13.domain-c35...}
TriggeredAlarmState : {alarm-37.host-37, alarm-37.host-39}
AlarmActionsEnabled : True
Tag                 : {}
Value               : {}
AvailableField      : {}
MoRef               : ClusterComputeResource-domain-c35
Client              : VMware.Vim.VimClient

[vSphere PowerCLI] C:\>
```

This variable is the base we are going to use to get the cluster configuration, now we are going to use it.

## Get the cluster resources summary

```powershell-interactive
[vSphere PowerCLI] C:\> $viewdl380.Summary

CurrentFailoverLevel : 0
AdmissionControlInfo : VMware.Vim.ClusterFailoverLevelAdmissionControlInfo
NumVmotions          : 5
TargetBalance        : 200
CurrentBalance       : 16
CurrentEVCModeKey    :
TotalCpu             : 63984
TotalMemory          : 77287268352
NumCpuCores          : 24
NumCpuThreads        : 48
EffectiveCpu         : 56784
EffectiveMemory      : 62981
NumHosts             : 2
NumEffectiveHosts    : 2
OverallStatus        : red
DynamicType          :
DynamicProperty      :

[vSphere PowerCLI] C:\>
```

## Get VMware HA configuration

```powershell-interactive
[vSphere PowerCLI] C:\> $viewdl380.Configuration

DasConfig       : VMware.Vim.ClusterDasConfigInfo
DasVmConfig     :
DrsConfig       : VMware.Vim.ClusterDrsConfigInfo
DrsVmConfig     :
Rule            : {sql-cl01, hpsim}
DynamicType     :
DynamicProperty :

[vSphere PowerCLI] C:\> $viewdl380.Configuration.DasConfig

Enabled                 : True
VmMonitoring            : vmMonitoringDisabled
HostMonitoring          : enabled
FailoverLevel           : 1
AdmissionControlPolicy  : VMware.Vim.ClusterFailoverLevelAdmissionControlPolicy
AdmissionControlEnabled : True
DefaultVmSettings       : VMware.Vim.ClusterDasVmSettings
Option                  : {das.isolationaddress1, das.isolationaddress2, das.usedefaultisolationaddress}
DynamicType             :
DynamicProperty         :

[vSphere PowerCLI] C:\>
```

## Get cluster advanced options

```powershell-interactive
[vSphere PowerCLI] C:\> $viewdl380.Configuration.DasConfig.Option

Key                                   Value                                 DynamicType                          DynamicProperty                     
---                                   -----                                 -----------                          ---------------                     
das.isolationaddress1                 192.168.126.2                                                                                                     
das.isolationaddress2                 192.168.126.250                                                                                                     
das.usedefaultisolationaddress        false                                                                                                          

[vSphere PowerCLI] C:\>
```

## Get DRS basic configuration

```powershell-interactive
[vSphere PowerCLI] C:\> $viewdl380.Configuration.DrsConfig

Enabled                   : True
EnableVmBehaviorOverrides : True
DefaultVmBehavior         : partiallyAutomated
VmotionRate               : 3
Option                    : {ForceAffinePoweron}
DynamicType               :
DynamicProperty           :

[vSphere PowerCLI] C:\> $viewdl380.Configuration.DrsConfig.Option

Key                                   Value                                 DynamicType                          DynamicProperty
---                                   -----                                 -----------                          ---------------
ForceAffinePoweron                    1

[vSphere PowerCLI] C:\>
```

## Get Virtual Port Groups of the cluster

```powershell-interactive
[vSphere PowerCLI] C:\> Get-View $viewdl380.Network | Select-Object Name

Name
----
HeartBeatClusterSQL
iSCSI-v101
Backend-v102
Management-v105

[vSphere PowerCLI] C:\>
```

## Get the Datastores configured in the cluster

```powershell-interactive
[vSphere PowerCLI] C:\> Get-View $viewdl380.Datastore | Select-Object Name

Name
----
FC_DS-DL380-01
FC_DS-DL380-02
iSCSI_DS-DL380-01
NFS_DS-DL380-01
LOCAL_DS-DL380-01
LOCAL_DS-DL380-02

[vSphere PowerCLI] C:\>
```

Finally to ease things at work I created a bunch of scripts that implement some of the above tasks, here it is an example. The syntax I used for all of them is `[script-name] <cluster-name>`.

```powershell
# Get-ClusterAdvancedOption.ps1
# PowerCLI script to get VMware HA advanced options of a given cluster
#
# Juan Manuel Rey - juanmanuel (dot) reyportal (at) gmail (dot) com
# http://jreypo.io
#
# Syntax: Get-ClusterAdvancedOption.ps1 &lt;cluster-name&gt;
#

param([string]$clustername)

$cluster = Get-Cluster $clustername
$viewcluster = Get-View $cluster.Id

$viewcluster.Configuration.DasConfig.Option
```

From this point I'll leave to you to investigate all the possibilities, as always if you know a better way to do it, have any question or want to say anything about the post please comment. I'm also preparing a new post with all my small scripts and hope to publish them next week.

Juanma.
