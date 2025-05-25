---
title: A bit of troubleshooting of the vCenter Server Appliance
date: 2012-06-28
type: post
classes: wide
published: true
status: publish
categories:
- Linux
- Sysadmin
- Virtualization
- VMware
tags:
- Linux
- sysadmin
- systems administration
- vCenter Server
- vCenter Server Appliancev
- VCSA
- VMware
- vSphere 5
author: juan_manuel_rey
comments: true
---

If your **vCSA** is configured to use the embedded DB2 database and if it’s not properly shutdown, next you power it on may be you should not be able to power on a VM like in the screenshot below.

[![](/assets/images/vm_power_not_allowed.png "Operation not allowed")]({{site.url}}/assets/images/vm_power_not_allowed.png)

Or the vSphere Client will not show some of information about the host or the VMs.

[![](/assets/images/vc_client_no_host_info.png)]({{site.url}}/assets/images/vc_client_no_host_info.png)

We all have seen those kind of errors in our homelab from time to time. In the Windows-based vCenter it was relatively easy to solve, close the client, log into the vCenter, restart the vCenter Server service and in the next login into the vSphere Client everything will go as expected.

However how can we resolve this issue in the vCenter Linux appliance? Can’t be easier.

There are two ways to restart the vCenter services in the vCSA:

- From he WebUI administration interface
- From the command line

For the first method log into the WebUI of the vCSA by accessing **https://vCSA_URL:5480** with your favorite web browser.

[![](/assets/images/vcsa_main_screen.png "vCSA main screen")]({{site.url}}/assets/images/image2.png)

In the **vCenter Server** screen go to the **Status** tab, stop and start the vCenter Server service using the Action buttons.

The second method is faster and easier, and to be sincere it feels more natural for me and probably for the other Unix Geek/Sysadmins out there.

The vCenter service in the Linux appliance is `vmware-vpxd` so with a simple `service vmware-vpxd restart` we’ll be on business again. Check the screenshot below.

[![](/assets/images/service_vpxd_running.png "Service VPXD restart and status")]({{site.url}}/assets/images/service_vpxd_running.png)

Finally as seen in the screen capture you can check the status of the service.

More on troubleshooting the vCSA in a future post.

Juanma.
