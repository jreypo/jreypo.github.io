---
layout: post
title: How to get iSCSI iqn using esxcli in ESXi 5
date: 2012-08-02
type: post
published: true
status: publish
categories:
- Storage
- Sysadmin
- Virtualization
- VMware
tags:
- esxcli
- ESXi
- ESXi5
- iqn
- iSCSI
- Storage
- sysadmin
- systems administration
- vSphere
- vSphere CLI
author: juan_manuel_rey
comments: true
---

Back in 2010 I wrote a post about [how to get the iSCSI iqn of an ESXi 4.x]({% post_url 2010-12-13-get-the-iscsi-iqn-of-an-esxi-using-the-cli %}) server using vSphere CLI from the vMA or any other system with vCLI installed on it.

The method described in that article is still valid for ESXi 5.0 since the old `vicfg` and `esxcfg` commands are still available, however with 5.0 version you can get a similar result using the new `esxcli` command and the corresponding namespaces, following is how to do it.

First task is to get a list of the iSCSI HBAs in order to know the name of the software iSCSI initiator.

[![](/images/esxcli_list_hbas.png "List host HBAs")]({{site.url}}/images/esxcli_list_hbas.png)

Next we get the info of the adapter.

[![](/images/get_adapter_info_esxcli.png "Get adapter information")]({{site.url}}/images/get_adapter_info_esxcli.png)

Look at the `Name` field to get the `iqn` and we are done.

Juanma.
