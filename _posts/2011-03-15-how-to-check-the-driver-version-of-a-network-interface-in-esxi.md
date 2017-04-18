---
layout: post
title: How to check the driver version of a network interface in ESX(i)
date: 2011-03-15
type: post
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- ESX
- esxcfg-module
- ESXi
- sysadmin
- systems administration
- vmkload_mod
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

Here are two quick ways to check the driver version of a network interface card. The commands must be executed in the ESX COS or ESX(i) Tech Support Mode.

-   `vmkload_mod`

`vmkload_mod` is a tool to manage VMkernel modules. It can be used to load and unload modules, list the loaded modules and get the general information and available parameters of each module.

```
~ # vmkload_mod -s bnx2x | grep Version
 Version: Version 1.54.1.v41.1-1vmw, Build: 260247, Interface: ddi_9_1 Built on: May 18 2010
~ #
```

-   `ethtool`

`ethtool` is a Linux command line tool that allow us to retrieve and modify the parameters of an ethernet device. It is present in the vast majority of Linux systems, including the ESX Service Console. Fortunately for us VMware has also included it within the busybox environment of the ESXi.

```
~ # ethtool -i vmnic0
driver: bnx2x
version: 1.54.1.v41.1-1vmw
firmware-version: BC:5.2.7 PHY:baa0:0105
bus-info: 0000:02:00.0
~ #
```

If you want to use PowerCLI for this task you should check Julian Wood ([@julian_wood](http://twitter.com/julian_wood)) [excellent post](http://www.wooditwork.com/2011/01/27/getting-nic-firmware-versions-with-powercli/) about it.

Juanma.
