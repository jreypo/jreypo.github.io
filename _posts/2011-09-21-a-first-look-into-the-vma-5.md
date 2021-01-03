---
title: A first look into the vMA 5
date: 2011-09-21
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
- vCenter Server
- vMA5
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

Like the rest of components of vSphere the vMA, vSphere Management Appliance, has been updated to the new version. In this post I will discuss the changes and features of the new vMA 5 and will show how to deploy and configure it.

First of all for those of you who have been hiding in a cave and know nothing about the vMA all you need to know for now is that is a virtual appliance provided by VMware that allows System Administrators to manage their virtual infrastructure and run scripts and agents that interact directly with the vCenter Server and ESXi. It can do it also without having to authenticate each time. I will not go deeper on this since there are tons of blog posts out there explaining it and also it's very well detailed in the vMA documentation.

### vMA 5 features and changes

The vMA 5 appliance is composed by the following elements:

-   **SUSE Linux Enterprise Server 11 SP1 64-bit**: This is a major change. Previous versions of the vMA were all based on Red Hat, either Red Hat Enterprise Linux or CentOS, but with the     introduction of vSphere 5 all virtual appliances have been migrated to SUSE Linux Enterprise Server 11.
-   **VMware Tools**
-   **vSphere CLI**
-   **vSphere SDK for Perl**
-   **Java JRE 1.6**
-   **vi-fastpass**: The authentication component of vMA.

As you can observe many of them are also present in former versions of the vMA.

Regarding the hardware requirements they are again very similar to vMA4.

-   ESXi host capable of running 64-bit guests
-   1 vCPU
-   3GB of storage space
-   512MB of RAM. This is recommended memory size, however vMA can be run with less RAM but its performance can be penalized.

The new vMA can be deployed from the vSphere Client connected to a **vCenter Server 5.0** or **vCenter Server 4.x** and can be run on the following vSphere releases:

-   vSphere 5.0
-   vSphere 4.1 and 4.1 Update 1
-   vSphere 4.0 Update 2

The systems that can be managed from the vMA are:

-   ESXi 3.5 Update 5
-   ESXi 4.0 Update 2
-   vSphere 4.1 and 4.1 Update 1
-   vSphere 5.0

### vMA 5 deployment and configuration

We are going to deploy the vMA 5 through the vSphere Client in the same manner as the vMA 4. Go to **File -> Deploy OVF** template and when the pop-up shows up browse for the vMA OVF and click next.

[![](/assets/images/ovf_deployment.png "vMA OVF deployment from vSphere Client")]({{site.url}}/assets/images/ovf_deployment.png)

Follow the screens until the last one to configure the datacenter and host or cluster where you want to deploy the vMA, configure the appliance to match your environment and click Finish to start the deployment.

[![](/assets/images/final_screen.png "Deployment final screen")]({{site.rul}}/assets/images/final_screen.png)

When the deployment is finished open the vMA virtual machine console and power it on. When the vMA boots for the first time it should be configured.

Once the OS is up the first prompt will ask for the network configuration.

[![](/assets/images/vma5_network_configuration.png "vMA5 network configuration")]({{site.url}}/assets/images/vma5_network_configuration.png)

In the next step you'll be asked for the password of vi-admin password. There is a major change here in comparison with the vMA 4, the vi-admin password must has an increased complexity and must contain at least:

-   Eight characters.
-   One upper case character.
-   One lower case character.
-   One numeral character.
-   One symbol.

The reason to this new password policy comes from the SUSE Enterprise Linux Server operating system the vMA 5 is based on. William Lamw ([@lamw](http://www.twitter.com/lamw)) provides the link to the Novell Knowledge Base article related to this topic in his excellent post **[Tips and tricks for the vMA 5** ](http://www.virtuallyghetto.com/2011/07/tips-and-tricks-for-vma-5.html).

[![](/assets/images/vi-admin_password.png "vi-admin user password")]({{site.url}}/assets/images/vi-admin_password.png)

Once network parameters and vi-admin password are configured the vMA should be ready to manage your vSphere servers and the console screen will appear.

The vMA5 can be managed from two ways:

-   Text-based console

[![](/assets/images/vma5_console.png "vMA5 console")]({{site.url}}/assets/images/vma5_console.png)

From the text-based console you can launch the initial configuration for the vMA networking, set the Timezone of the vMA and login into the Linux command-line interface like in the previous releases of the vMA to manage the appliance from the Linux shell and of course to manage your vSphere infrastructure. As always SSH access is also allowed to the lastone.

-   Browser-based Web UI

[![](/assets/images/vma5_web_login.png "vMA5 web interface")]({{site.url}}/assets/images/vma5_web_login.png)

The Web UI enables you only to manage the vMA itself and not the vCenter and ESX(i) servers. To access the Web UI point your browser to **https://<vma_address_or_hostname>:5480** and login as vi-admin user. From there you can do the following tasks:

-   Check the status of the appliance, set the timezone and perform a system reboot or shutdown

[![](/assets/images/vma5_web_system.png "vMA5 Web UI System screen")]({{site.url}}/assets/images/vma5_web_system.png)

-   Manage the appliance network and proxy server settings

[![](/assets/images/vma5_web_network.png "vMA5 Web UI Network screen")]({{site.url}}/assets/images/vma5_web_network.png)

-   Update the vMA 5

[![](/assets/images/vma5_web_update.png "vMA5 Web UI Update screen")]({{site.url}}/assets/images/vma5_web_update.png)

This last option is of significance since now this is the way to update the vMA because the vma-update utility has been removed.

Juanma.
