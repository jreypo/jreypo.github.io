---
title: How to patch your VIO environment
date: 2015-04-29
type: post
classes: wide
published: true
status: publish
categories:
- OpenStack
- VMware
tags:
- Cloud
- OpenStack
- SDDC
- VIO
- VMware
- VMware Integrated OpenStack
author: juan_manuel_rey
comments: true
image:
  feature: openstack-banner.jpg
---

[VMware](http://www.vmware.com) has released the first patch for **VMware Integrated OpenStack**. This patch release comes with improvements around the installer, Keystone service and fixes some security issues. Review the [Release Notes](https://www.vmware.com/support/integrated-openstack/doc/vmware-integrated-openstack-101-release-notes.html) to get full details of what is included.

After the patch was released I thought it was a perfect time to upgrade my VIO lab, document the procedure and publish it, so without further ado lets get some patches installed!

## Step 1 - Upload and install the patch

Get the patch from [VIO product download page](https://my.vmware.com/group/vmware/info?slug=datacenter_cloud_infrastructure/vmware_integrated_openstack/1_0), of course you need to have the proper rights to do it, the patch is a Debian package in `deb` format. There are some caveats here, the way to upload and install the patch is using vSphere Web Client from **Manage ->  Updates**.

[![](/assets/images/screen-shot-2015-04-28-at-23-38-55.png)]({{site.url}}/assets/images/screen-shot-2015-04-28-at-23-38-55.png)

However after the immediate release of the patch an issue was identified using this method and currently until it is solved the safest way to do it is using the CLI. Use your favorite SCP/SFTP client to upload the patch to VIO Management Server as `viouser`.

Add the patch using `viopatch` command.

```
viouser@vio-manager:~$ sudo viopatch add -l /home/viouser/vio-patch-1_1.0.1.2668568_all.deb
[sudo] password for viouser:
/home/viouser/vio-patch-1_1.0.1.2668568_all.deb patch has been added.
viouser@vio-manager:~$
```

List the patches to verify that has been added.

```
viouser@vio-manager:~$ viopatch list
Name         Version        Type    Installed
-----------  -------------  ------  -----------
vio-patch-1  1.0.1.2668568  infra   No

viouser@vio-manager:~$
```

Install the patch, before installing verify that VIO Cluster is in Running status or the update will fail. The patch can also be applied before deploying OpenStack.

```
viouser@vio-manager:~$ sudo viopatch install --patch vio-patch-1 --version 1.0.1.2668568
[sudo] password for viouser:
Installing patch vio-patch-1 version 1.0.1.2668568
done
Installation complete for patch vio-patch-1 version 1.0.1.2668568
viouser@vio-manager:~$
```

## Step 2 - Verify the installation

Log out and log in back in vSphere Web Client. The new version and build number can be verified in the Summary tab.

[![](/assets/images/screen-shot-2015-04-29-at-01-09-21.png)]({{site.url}}/assets/images/screen-shot-2015-04-29-at-01-09-21.png)

Also in **Manage -> Updates** the newly installed patch can be seen in more detail.

[![](/assets/images/screen-shot-2015-04-29-at-01-09-53.png)]({{site.url}}/assets/images/screen-shot-2015-04-29-at-01-09-53.png)

Ans this is it. Anyone that has ever to endure the pain of patching an OpenStack installation, either lab or production environment, I am sure that will appreciate how clean and easy is the process in VIO.

Juanma.
