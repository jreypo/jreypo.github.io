---
title: DNS configuration with esxcli
date: 2011-10-17
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- esxcli
- ESXi5
- networking
- sysadmin
- systems administration
- Virtualization
- VMware
- vSphere
showComments: true
---

With release of ESXi 5.0 the `esxcli` command has been also vastly improved. One of this new capabilities is the possibility to manage the DNS configuration of the server.

The basic syntax for dns is:

```text
~# esxcli network ip dns
```

This gives you two namespaces to work with:

- `search`
- `server`

[![](/images/esxcli_dns1.png)](/images/esxcli_dns1.png)

With the first one you can manage the suffixes for DNS search and the second is for the DNS server to be used by the ESXi.

## Server operations

[![](/images/image.png)](/images/image.png)

Add a new server:

[![](/images/image1.png)](/images/image1.png)

Remove a configured server:

[![](/images/image2.png)](/images/image2.png)

## Domain search operations

List configured domain suffixes:

[![](/images/image3.png)](/images/image3.png)

Add a new domain:

[![](/images/image4.png)](/images/image4.png)

Remove a configured domain:

[![](/images/image5.png)](/images/image5.png)

Juanma.
