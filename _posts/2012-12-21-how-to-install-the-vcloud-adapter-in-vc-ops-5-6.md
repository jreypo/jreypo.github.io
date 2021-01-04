---
title: How to install the vCloud Adapter in vCenter Operations 5.6
date: 2012-12-21
type: post
classes: wide
published: true
status: publish
categories:
- Virtualization
- VMware
tags:
- vC Ops
- vCenter Operations
- vCenter Operations Manager
- vCloud Adapter
- vCloud Director
- VMware
author: juan_manuel_rey
comments: true
---

Installing the vCloud Director adapter, or any vCenter Operations adapter, is a relatively easy task as we will explain in this post.

Firstly you need to download the adapter from [ftp.integrien.com](ftp://ftp.integrien.com). Choose the `.PAK` file.

[![](/assets/images/vcops_vcd_adapter_pak_file.png "vCloud Adapter PAK file")]({{site.url}}/assets/images/vcops_vcd_adapter_pak_file.png)

Once it is downloaded log into vC Ops Admin UI **https://vcops-ui-vm/admin**. From the **Update** tab browse for the downloaded file and click **Update**.

[![](/assets/images/install_vcops_vcd_adapter.png "Install vCloud adapter")]({{site.url}}/assets/images/install_vcops_vcd_adapter.png)

It will ask for confirmation and the will present you the EULA. Accept it and click **OK**.

[![](/assets/images/vcops_vcd_accept_eula.png "Accept EULA")]({{site.url}}/assets/images/vcops_vcd_accept_eula.png)

Will ask for confirmation again.  During the updating process you will be automatically logged out from the Administration Portal, will be unable to log back in until the update is done.

The update process can take a few minutes so grab a coffee and wait until it’s done.

[![](/assets/images/vcd_adapter_updating_progress.png "Updating progress")]({{site.url}}/assets/images/vcd_adapter_updating_progress.png)

Once the vCloud Adapter installation is done proceed to vCenter Operations Custom UI **https://vcops-ui-vm/custom**. From the **Admin** menu go to **Support**.

[![](/assets/images/vcops_ui_admin_support.png)]({{site.url}}/assets/images/vcops_ui_admin_support.png)

In the support screen open the **Info** tab, look for the Adapters Info pane and click the **Describe** gear button.

[![](/assets/images/vcops_describe_adapter.png "Click Describe")]({{site.url}}/assets/images/vcops_describe_adapter.png)

When the describe process is finished refresh the page to check for the adapter presence. Match the adapter version with the build number of the downloaded package.

[![](/assets/images/vcops_vcd_adapter_version.png "Verify adapter version")]({{site.url}}/assets/images/vcops_vcd_adapter_version.png)

At this point the adapter is installed, now we are going to configure it to collect from our vCloud instance.

From the **Environment** menu select **Configuration -> Adapter Instances**. Select the vC Ops Server collector and the vCloud adapter from the drop down menus. Click on **Add New Adapter Instance**.

[![](/assets/images/vcops_vcd_add_new_adapter.png "Add new Adapter Instance")]({{site.url}}/assets/images/vcops_vcd_add_new_adapter.png)

Fill out he fields from the pop-up window. For the IP/Hostname field the public address of the vCloud can be used if a REST API base URL has been assigned.

[![](/assets/images/vcops_vcd_enter_adapter_info.png "Enter adapter instance info")]({{site.url}}/assets/images/vcops_vcd_enter_adapter_info.png)

Remember to set the **Auto Discovery** option to true unless you want to force the discovery manually.

At this point there will be no credential available from the drop down menu. Click **Add** to create it.

[![](/assets/images/vcops_vcd_adapter_credentials.png "Enter vCloud adapter credentials")]({{site.url}}/assets/images/vcops_vcd_adapter_credentials.png)

Click OK and the vCloud Adapter should be configured and collecting. To test it go to **Environment –> Environment Overview**. In the left pane look for vCloud related **Resource Kind**, select anyone and look for new resources with a vCloud Data Source in the right pane.

[![](/assets/images/vcd_adapter_objects.png "vCloud Adapter objects")]({{site.url}}/assets/images/vcd_adapter_objects.png)

Following this procedure any vCenter Operations Manager relationship adapter can be installed and almost configure, have in mind that of course there will be differences in the credentials and the adapter specifics.

Juanma.
