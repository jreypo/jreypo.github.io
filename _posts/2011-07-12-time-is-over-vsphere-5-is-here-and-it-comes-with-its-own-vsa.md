---
title: Time is over... vSphere 5 is here and it comes with its own VSA
date: 2011-07-12
type: post
classes: wide
published: true
status: publish
categories:
- Virtualization
- VMware
tags:
- ESXi
- ESXi5
- P4000 VSA
- vCenter Server
- Virtualization
- VMware
- VSA
- vSphere
- vSphere 5
author: juan_manuel_rey
comments: true
---

Today has been a great day for the people of **VMware**, they presented [**vSphere 5**](http://www.vmware.com/products/vsphere/overview.html) in a huge online event and a lot of new interesting features have been finally unveiled.

-   Storage DRS.
-   Profile-Driven Storage.
-   VAAI v2.
-   vSphere Auto Deploy.
-   Software FCoE initiator.
-   Up to 32 vCPUs and 1TB of RAM per virtual machine.
-   VMFS5.
-   New vSphere HA framework.
-   vSphere Web Client (we've been expecting something like this for Linux ad Mac OS X)
-   vCenter Server Appliance. On Linux!
-   New vSphere Storage Appliance.

All of them are incredibly cool features however there is one that instantly got my attention:

**"VSA - VMware vSphere Storage Appliance"** a virtual appliance that will allow to turn the DAS storage of the server into shared storage in those customers with no budget to purchase a full working SAN or where the performance is not a key factor. Wait a minute this sounds familiar.

This is not a new feature. In fact HP, and Lefthand Networks before it was acquired by HP, and VMware have been provided this feature for years through the [HP P4000 VSA](http://h18006.www1.hp.com/products/storage/software/vsa/index.html). After HP/Lefthand other vendors, like NetApp, have released their own VSAs, not as feature rich as the HP appliance of course ;-)

This is an interesting movement by VMware but I think it has its cons. The VSA is not included by default with the vSphere software, you have to pay for it, this is not bad of course since you also have to pay for the P4000 VSA.

That said, what is the advantage of running the VMware provided VSA instead of the HP one? The only one I can think of is the integration of the management interface with the vCenter Server and vSphere Client but with the [HP Insight Control for VMware vCenter Server](http://h18000.www1.hp.com/products/servers/management/integration.html) you can also manage, at least in part, your VSAs and I believe the HP CMC interface is rich-feature enough to use it.

Anyway, I'm dying to get my hands on this storage appliance and test it against my beloved P4000 VSA. In the meantime a nice introduction and a couple of videos of the product can be found [here](http://www.vmware.com/products/datacenter-virtualization/vsphere/vsphere-storage-appliance/overview.html).

Like Calvin ([@HPStorageGuy](http://twitter.com/#%21/hpstorageguy)) said today in a tweet "Welcome to the past" ;-)

Juanma.
