---
title: VCSA 5.1– A quick look
date: 2012-08-28
type: post
classes: wide
published: true
status: publish
categories:
- Virtualization
- VMware
tags:
- vCenter Server
- vCenter Server Appliance
- vCloud Suite
- VCSA
- VMware
- vSphere
- vSphere 5.1
author: juan_manuel_rey
comments: true
---

Yesterday was a very exciting day, **VMware** finally announced the new vCloud Suite 5.1. With new products and features announced, each one of them as great as ever.

But yours truly decided that instead of presenting you a generic "What’s new” post it was more interesting, at least for me and hope also to you, to focus in one of my favorite pieces of vSphere, the **vCenter Server Virtual Appliance** and in a couple of articles describe what new features are coming and how to perform an upgrade from 5.0 to 5.1.

#### What’s new in the VCSA 5.1

With this new release of the **vCenter Server VA** a few new features have been added, most of them related to the WebUI and to the new features and services that have been released globally for the vCenter 5.1, Windows and Linux based versions.

These new enhancements can be seen in the Summary section of the **vCenter Server Tab** as shown in the screenshot below.

[![](/assets/images/vcsa_summary.png "vCSA Summary")]({{site.url}}/assets/images/vcsa_summary.png)

From here all the vCenter Service can be started and stopped. The Services Tab no longer exists and it’s now a section of the vCenter Server Tab.

In the Storage Usage are you can check the disk usage of the different components of the VCSA.

There is also a Utilities are where you can:

-   Generate a support bundle in case of an incidence with the vCenter Server.
-   Download the configuration file of the VCSA and then open it in your favorite text editor.

[![](/assets/images/vcsa_config_file.png "vCSA configuration file")]({{site.url}}/assets/images/vcsa_config_file.png)

-   Launch again the Setup Wizard to configure the VCSA from scratch or make modifications to some of he initial settings.
-   Upload the Windows Sysprep files to the vCenter VA.

For this last option is quite simple, just click the **Upload** button.

[![](/assets/images/vcsa_upload_sysprep_files.png "Upload Sysprep files")]({{site.url}}/assets/images/vcsa_upload_sysprep_files.png)

A new windows will open, select the operating system and browse for the location of the files.

[![](/assets/images/sysprep_files_upload_screen.png "Select Sysprep files OS")]({{site.url}}/assets/images/sysprep_files_upload_screen.png)

The files will be uploaded to `/etc/vmware-vpx/sysprep/<OS>`.

Additionally in the vCenter Server Tab two new sections have been added.

The **Services** section where the VCSA administrator can setup:

-   ESXi Dump Collector repository size
-   vSphere Auto Deploy repository size
-   Inventory size

[![](/assets/images/vcsa_vc_services.png "vCenter Services")]({{site.url}}/assets/images/vcsa_vc_services.png)

The **SSO section**. Here you can configure all the settings related to the newly introduced Single Sign On server.

[![](/assets/images/vcsa_sso_settings.png "vCSA SSO settings")]({{site.url}}/assets/images/vcsa_sso_settings.png)

In the next post we will discuss about VCSA 5.1 initial deployment and how to upgrade from 5.0 and 5.1.

Juanma.
