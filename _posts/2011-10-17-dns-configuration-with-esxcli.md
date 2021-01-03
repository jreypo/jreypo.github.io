---
title: DNS configuration with esxcli
date: 2011-10-17
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
- ESXi5
- networking
- sysadmin
- systems administration
- Virtualization
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

With release of ESXi 5.0 the `esxcli` command has been also vastly improved. One of this new capabilities is the possibility to manage the DNS configuration of the server.

The basic syntax for dns is:

```
~# esxcli network ip dns
```

This gives you two namespaces to work with:

-   `search`
-   `server`

[![](/assets/images/esxcli_dns1.png)]({{site.url}}/assets/images/esxcli_dns1.png)

With the first one you can manage the suffixes for DNS search and the second is for the DNS server to be used by the ESXi.

### Server operations

[![](/assets/images/image.png)]({{site.url}}/assets/images/image.png)

Add a new server:

[![](/assets/images/image1.png)]({{site.url}}/assets/images/image1.png)

Remove a configured server:

[![](/assets/images/image2.png)]({{site.url}}/assets/images/image2.png)

### Domain search operations

List configured domain suffixes:

[![](/assets/images/image3.png)]({{site.url}}/assets/images/image3.png)

Add a new domain:

[![](/assets/images/image4.png)]({{site.url}}/assets/images/image4.png)

Remove a configured domain:

[![](/assets/images/image5.png)]({{site.url}}/assets/images/image5.png)

Juanma.
