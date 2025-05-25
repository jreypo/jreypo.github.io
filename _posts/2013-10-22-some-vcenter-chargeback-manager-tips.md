---
title: Some vCenter Chargeback Manager tips
date: 2013-10-22
type: post
classes: wide
published: true
status: publish
categories:
- Virtualization
- VMware
tags:
- CBM
- Chargeback Manager
- vCenter Chargeback
- VMware
author: juan_manuel_rey
comments: true
---

After the [previous article]({% post_url 2013-10-09-regenerate-chargeback-manager-ssl-certificate %}) about SSL certificate generation in Chargeback I decided that it was worth to write a couple more tips in a second blog post. This is not a “how to install…” or “how to configure…” post. There are other bloggers in the community that have written about it before so I believe there is no point on repeating the same.

## Database configuration

At one point the CBM installer will ask for the database details, apparently nothing to worry about. Except for the following:

- **Port:** It says is optional but if you don’t enter the port the connection will fail. The default port is 1433 but of course fill it with the value from your installation.

[![](/assets/images/cbm_db_config.png "Chargeback database configuration")]({{site.url}}/assets/images/cbm_db_config.png)

## Adding a vCenter Server

The first task you want to perform after Chargeback installation is to add a vCenter Server, in theory pretty easy until you get to the database screen. It is very important to remember that in the **Database URL** field you only need to put the database server IP address as shown in the screen capture.

[![](/assets/images/cbm_vc_conifig.png "vCenter database URL")]({{site.url}}/assets/images/cbm_vc_conifig.png)

Juanma.
