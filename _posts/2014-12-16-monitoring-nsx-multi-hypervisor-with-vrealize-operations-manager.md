---
layout: post
title: Monitoring NSX Multi-Hypervisor with vRealize Operations Manager
date: 2014-12-16
type: post
published: true
status: publish
categories:
- DevOps
- Networking
- Sysadmin
- VMware
tags:
- network virtualization
- networking
- NSX
- NSX-MH
- vC Ops
- vCenter Operations
- vCenter Operations Manager
- VMware
- vRealize Operations
- vROps

author: juan_manuel_rey
comments: true
---

VMware has released a new **vRealize Operations Manager** management pack for **NSX Multi-hypervisor**. This new management pack will allow vROps to extend its management capabilities into any NSX-MH infrastructure.

This management pack provides a great set a features, including:

-   Operational visibility into the different NSX-MH components, from NSX Manager to Controllers, transport nodes and logical elements of the network.
-   Search and drill down functionality to help the administrator monitor the health of the NSX objects.
-   Alerts and root cause problem solving capabilities by detecting configuration, connectivity and health deficiencies into the NSX environment.
-   Report templates for NSX Multi-Hypervisor environment.

The management pack requires vRealize Operations Manager 6.0 and can be downloaded from [VMware Solutions Exchange](https://solutionexchange.vmware.com/store/products/management-pack-for-nsx-for-multi-hypervisor-1-0#.VI9wfIrF-n0).

### Installation

To install this management pack go to **Administration** in the left pane.

[![](/images/screen-shot-2014-12-16-at-01-15-10.png)](https://jreypo.files.wordpress.com/2014/12/screen-shot-2014-12-16-at-01-15-10.png)

From there go to **Solutions** and on the right pane click on the plus sign to add the new management pack.

[![](/images/screen-shot-2014-12-16-at-01-15-24.png?w=580)](https://jreypo.files.wordpress.com/2014/12/screen-shot-2014-12-16-at-01-15-24.png)

Browse for the pack installation file, click **Upload** and then click **Next** when the installation file is uploaded.

[![](/images/screen-shot-2014-12-16-at-01-16-27.png?w=580)](https://jreypo.files.wordpress.com/2014/12/screen-shot-2014-12-16-at-01-16-27.png)

Accept the EULA and proceed to the last screen. Wait until the management pack is installed and then click **Finish**.

[![](/images/screen-shot-2014-12-16-at-01-19-10.png?w=580)](https://jreypo.files.wordpress.com/2014/12/screen-shot-2014-12-16-at-01-19-10.png)

### Configure the adapter instance

The first task is to create the credentials for the solution. Access **Administration -> Credentials** and create a new credential for the **NSX-MH Adapter**.It has to include the administration credentials for the NSX Controller, NSX Manager and vCenter Server.

[![](/images/screen-shot-2014-12-16-at-02-19-17.png)](https://jreypo.files.wordpress.com/2014/12/screen-shot-2014-12-16-at-02-19-17.png)

Next access **Administration -> Solutions**, select the NSX-MH pack and click on the gear icon.

[![](/images/configure-nsx-mh.png?w=580)](https://jreypo.files.wordpress.com/2014/12/configure-nsx-mh.png)

On the pop-up window enter the IP address or the FQDN for:

-   NSX Controller
-   NSX Manager
-   vCenter Server

Only the first NSX Controller is needed.

[![](/images/configure-nsx-mh_2.png?w=580)](https://jreypo.files.wordpress.com/2014/12/configure-nsx-mh_2.png)

Test the connection, accept the certificates for the different components and click Save Settings. After this the adapter is configured and will start collecting data, it will take a some time until it displays data, depending on the size of the NSX environment, to have a full collection of data.

### NSX-MH dashboards

Out of the box the management pack comes with three dashboards.

#### NSX-MH Main

It provides an overview of the health of the different network objects

[![](/images/screen-shot-2014-12-16-at-01-29-26.png?w=580)](https://jreypo.files.wordpress.com/2014/12/screen-shot-2014-12-16-at-01-29-26.png)

#### NSX-MH Topology

Provides details about the topology of a selected object, how it connects in the networks and a view of the related alerts and metrics.

[![](/images/screen-shot-2014-12-15-at-02-30-37.png?w=580)](https://jreypo.files.wordpress.com/2014/12/screen-shot-2014-12-15-at-02-30-37.png)

#### NSX-MH Object Path

This dashboard enables the administrator to visually depict a the path between two selected objects and verify how they are connected between each other and other objects.

[![](/images/screen-shot-2014-12-16-at-01-32-14.png?w=580)](https://jreypo.files.wordpress.com/2014/12/screen-shot-2014-12-16-at-01-32-14.png)

Juanma.

 
