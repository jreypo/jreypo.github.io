---
layout: post
title: Change ESXi 5 iSCSI iqn with esxcli
date: 2012-08-02
type: post
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- esxcli
- ESXi
- ESXi5
- iqn
- iSCSI
- sysadmin
- systems administration
- vSphere
- vSphere CLI
author: juan_manuel_rey
comments: true
---

After my previous post about [getting the `iqn` of an ESXi using `esxcli`]({% post_url 2012-08-02-how-to-get-iscsi-iqn-using-esxcli-in-esxi-5 %}) Andy Banta ([@andybanta](http://twitter.com/andybanta)) commented on Twitter that you can also change the `iqn` of the host with `esxcli`.

As he said it would be tremendously useful if you need to physically replace the server and don’t want to modify all your storage infrastructure, it’s easier to just modify the `iqn` of the new server and set it to the old name.

The task is as easier as the one described in last post. Using `esxcli` command with the `iscsi` namespace you can change the name and the alias of the adapter.

[![](/images/esxcli_iscsi_namespaces.png "esxcli iSCSI namespace")]({{site.url}}/images/esxcli_iscsi_namespaces.png)

As a precaution first retrieve the current `iqn` to check that it’s the correct server.

[![](/images/get_iscsi_iqn_esxcli_vcli.png "Retrieve current iqn")]({{site.url}}/images/get_iscsi_iqn_esxcli_vcli.png)

To change the name you have to provide the adapter and the new name.

[![](/images/set_iscsi_iqn_esxcli_vcli.png "Set new iSCSI iqn")]({{site.url}}/images/set_iscsi_iqn_esxcli_vcli.png)

Hope you find this useful, any comments and suggestions are welcome as always.

Juanma.
