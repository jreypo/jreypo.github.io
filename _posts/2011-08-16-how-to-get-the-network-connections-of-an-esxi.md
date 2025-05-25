---
title: How to get the network connections of an ESXi
date: 2011-08-16
type: post
classes: wide
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- esxcli
- ESXi
- ESXi4
- networking
- sysadmin
- systems administration
- vMA
- VMware
- vSphere
- vSphere CLI
author: juan_manuel_rey
comments: true
---

We are going to suppose that you are trying to troubleshoot your ESXi network problems and as an experienced sysadmin one of the first things to do is getting the network connections of the host. But you are in ESXi and that means there is no `netstat` command, that handy Unix command that saved your life so many times in the past.

Please don't panic yet, because as always in VMware there is a solution for that: `esxcli` to the rescue. Here it is the way to list the network connections of your ESXi host, both for **ESXi 4.1** and **ESXi 5**.

## ESXi 4.1

I tested it in ESXi 4.1 and ESXi 4.1 Update 1. The `network` namespace is not available in ESXi 4.0.

[![](/assets/images/esxi4.png "ESXi 4.1 Update 1 network connections")]({{site.url}}/assets/images/esxi4.png)

## ESXi 5

[![](/assets/images/esxi51.png "ESXi 5 network connections")]({{site.url}}/assets/images/esxi51.png)

I used Remote Tech Support (SSH), simply known as SSH in ESXi5, in both examples but you can also launch the command from the vMA or using vSphere CLI from a Windows or a Linux machine.

## vMA 4.1

```
[vi-admin@vma ~]$ esxcli --server=arrakis.jreypo.local --username=root network connection list
```

## vMA 5

```
vi-admin@vma5:~> esxcli --server=esxi5.jreypo.local --username=root network ip connection list
```

Juanma.
