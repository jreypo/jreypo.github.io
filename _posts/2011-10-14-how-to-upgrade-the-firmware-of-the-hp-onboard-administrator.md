---
title: How to upgrade the firmware of the HP Onboard Administrator
date: 2011-10-14
type: post
classes: wide
published: true
status: publish
categories:
- HP
tags:
- HP
- HP BladeSystem
- HP c-Class enclosures
- HP Onboard Administrator
author: juan_manuel_rey
comments: true
---

A post mostly for documentation sake on how to upgrade the firmware of the [Onboard Administrator](http://h18004.www1.hp.com/products/blades/components/onboard/index.html?jumpid=reg_R1002_USEN)
of an [HP BladeSystem](http://h18004.www1.hp.com/products/blades/bladesystem/index.html) enclosure.

The update is a very straight forward process that can be done entirely through the OA web administration interface.

Go to **Active Onboard Administrator –> Firmware Update**.

[![](/assets/images/oa1.png "OA Firmware Update")]({{site.url}}/assets/images/oa1.png)

From that screen you can upload the firmware from your system or enter an URL where the image is available. For our example we are going to use the upload option. Browse to the firmware image file and click **Upload**.

The OA web administration interface will logout any connected user and will start the upgrading process.

[![](/assets/images/oa2.png "OA2")]({{site.url}}/assets/images/oa2.png)

During the upgrade a progress bar like the one in the above screenshot will be shown and the OA will be reset when the process is complete.

[![](/assets/images/oa3.png "OA3")]({{site.url}}/assets/images/oa3.png)

Finally log into the OS and from the **Active Onboard Administrator** screen check the OA version.

[![](/assets/images/oa4.png "OA4")]({{site.url}}/assets/images/oa4.png)

Juanma.
