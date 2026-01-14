---
title: How to get ESXi network connections using PowerCLI
date: 2011-09-07
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
showComments: true
---

In a previous [post]({{< ref "posts/2011-08-16-how-to-get-the-network-connections-of-an-esxi.md" >}}) I described how to get the network connections of an ESXi server using `esxcli` from [Tech Support Mode](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1017910) and [vSphere CLI](http://www.vmware.com/support/developer/vcli/). Following I'll show you how to get the same information from an ESXi 4.1 and 5.0 using [PowerCLI](http://communities.vmware.com/community/vmtn/server/vsphere/automationtools/powercli).

The key to perform this tasks the `Get-EsxCli` cmdlet. This command was introduced with PowerCLI 4.1.1 and its purpose is to expose `esxcli` framework.

The first task to do with `Get-EsxCli` is to create a wrapper using a variable that will expose `esxcli` functionality.

[![](/images/get-esxcli.png "Get-EsxCli")](/images/get-esxcli.png)

As it can be seen in the screenshot, all the namespace of my whitebox are exposed just like with `esxcli` command. Now we are going to get the network connections of the host.

[![](/images/esxi4_net-connection.png "ESXi4 network connections")](/images/esxi4_net-connection.png)

Finally following is the syntax to get the network connections of an ESXi 5 server.

[![](/images/esxi5_net-connection.png "ESXi5 network connections")](/images/esxi5_net-connection.png)

In both cases I used the `Format-Table` cmdlet to get the output in a easily readable and useful format.

Juanma.
