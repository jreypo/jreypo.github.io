---
layout: post
title: Retrieving ESX(i) hardware info with PowerCLI
date: 2011-06-07
type: post
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- ESXi
- PowerCLI
- Powershell
- sysadmin
- systems administration
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

We all know how to look for the basic hardware info of an ESX(i) server, just open the vSphere Client go to **Summary** tab and you will presented with the familiar summary screen that includes **General** section with the manufacturer, model, etc.

[![](/images/general.png "General ESXi info")]({{site.url}}/images/general.png)

We can also enter the **Configuration** tab and get a bit more detailed information for each hardware component.

But why we have to enter a graphical interface in order to get several pieces of information from different areas of the GUI when we can have all that information just a few commands away if we use PowerCLI for the task.

{% highlight text %}
[vSphere PowerCLI] C:\Users\jreypo> $bl46001 = Get-VMHost esxbl460-01
[vSphere PowerCLI] C:\Users\jreypo>
[vSphere PowerCLI] C:\Users\jreypo> $viewbl460 = Get-View $bl46001.Id
[vSphere PowerCLI] C:\Users\jreypo> $viewbl460.Summary
[vSphere PowerCLI] C:\Users\jreypo> $viewbl460.Summary.Hardware

Vendor               : HP
Model                : ProLiant BL460c G7
Uuid                 : 36303332-3531-435a-4a30-333931304b4c
OtherIdentifyingInfo :
MemorySize           : 103067881472
CpuModel             : Intel(R) Xeon(R) CPU           X5670  @ 2.93GHz
CpuMhz               : 2933
NumCpuPkgs           : 2
NumCpuCores          : 12
NumCpuThreads        : 24
NumNics              : 8
NumHBAs              : 3
DynamicType          :
DynamicProperty      :

[vSphere PowerCLI] C:\Users\jreypo>
{% endhighlight %}

Oh yes! I love PowerCLI :-)

Juanma.
