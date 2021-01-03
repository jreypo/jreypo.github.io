---
title: How to get ESX(i) network connections using PowerCLI
date: 2011-09-07
type: post
classes: wide
published: true
status: publish
categories:
- Sysadmin
- VMware
- Windows
tags:
- esxcli
- ESXi4
- ESXi5
- Get-EsxCli
- PowerCLI
- Powershell
- sysadmin
- systems administration
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

In a previous [post]({% post_url 2011-08-16-how-to-get-the-network-connections-of-an-esxi %}) I described how to get the network connections of an ESXi server using `esxcli` from [Tech Support Mode](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1017910) and [vSphere CLI](http://www.vmware.com/support/developer/vcli/). Following I'll show you how to get the same information from an ESXi 4.1 and 5.0 using [PowerCLI](http://communities.vmware.com/community/vmtn/server/vsphere/automationtools/powercli).

The key to perform this tasks the `Get-EsxCli` cmdlet. This command was introduced with PowerCLI 4.1.1 and its purpose is to expose `esxcli` framework.

The first task to do with `Get-EsxCli` is to create a wrapper using a variable that will expose `esxcli` functionality.

[![](/assets/images/get-esxcli.png "Get-EsxCli")]({{site.url}}/assets/images/get-esxcli.png)

As it can be seen in the screenshot, all the namespace of my whitebox are exposed just like with `esxcli` command. Now we are going to get the network connections of the host.

[![](/assets/images/esxi4_net-connection.png "ESXi4 network connections")]({{site.url}}/assets/images/esxi4_net-connection.png)

Finally following is the syntax to get the network connections of an ESXi 5 server.

[![](/assets/images/esxi5_net-connection.png "ESXi5 network connections")]({{site.url}}/assets/images/esxi5_net-connection.png)

In both cases I used the `Format-Table` cmdlet to get the output in a easily readable and useful format.

Juanma.
