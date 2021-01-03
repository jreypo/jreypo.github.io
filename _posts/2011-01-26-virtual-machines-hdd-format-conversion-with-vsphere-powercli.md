---
title: Virtual machines HDD format conversion with vSphere PowerCLI
date: 2011-01-26
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
- Storage
- sysadmin
- systems administration
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

What I like the most of **PowerCLI** is the power that gives you, instead of relaying on a GUI every second you just issue a couple of commands and everything gets done. Of course thin provisioning is no exception to this and following is the way to convert from thin to thick a VM hard disk and vice versa.

The first task is to get the type of the hard disks. You can use the vSphere Client to check the format of the disk, just edit the VM setting and go the the hard disk as shown in the screenshot below.

[![](/assets/images/format-vsphereclient.png "HDD format check with vSphere Client")]({{site.url}}/assets/images/format-vsphereclient.png)

However using the vSphere Client is slow since you can only check one VM at a time, fortunately for us PowerCLI can offer a clean and elegant way to retrieve type for every disk of every virtual machine.

[![](/assets/images/get-harddisks2.png "Get-HardDisks")]({{site.url}}/assets/images/get-harddisks2.png)

Next we are going to convert from thin to thick the hard disk of the *debian-vm1* virtual machine. To do so we are going to use the cmdlet `Set-HardDisk`.

[![](/assets/images/inflate-disks.png "Disk conversion")]({{site.url}}/assets/images/inflate-disks.png)

Now that the disk is converted, how can we revert it using only PowerCLI? The way to do it is to move the disk to another datastore and convert it during the process.

The cmdlet to use is again `Set-HardDisk` with the `-Datastore` and `-StorageFormat` options. Take into account that these options can only be used if you are connected to the vCenter Server, you can't do this when directly connected to the ESX(i) server.

[![](/assets/images/thin-to-thick.png "Thick to thin conversion")]({{site.url}}/assets/images/thin-to-thick.png)**

Any other methods to execute the same operation are always welcome so please comment.

Juanma.
