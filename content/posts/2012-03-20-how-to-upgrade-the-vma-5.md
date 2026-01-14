---
title: How to upgrade the vMA 5
date: 2012-03-20
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- sysadmin
- systems administration
- Virtualization
- vMA
- vMA5
- VMware
- vSphere
- vSphere 5
showComments: true
---

Last week **[vSphere 5 Update 1](http://blogs.vmware.com/vsphere/2012/03/cloud-infrastructure-suite-updates.html)** was released by VMware, along with the main products some of the SDKs and automation tools were also updated, including the vMA appliance.

As you should remember from my [first post]({{< ref "posts/2011-09-21-a-first-look-into-the-vma-5.md" >}}) about **vMA 5** the classic `vma-update` utility is no longer available. So to be able to update our vMA to the new version we have to use the Web UI. Following is the procedure to perform the upgrade.

First access the web interface using `vi-admin` user as always.

[![](/images/vma_vima_login.png "vMA login screen")](/images/vma_vima_login.png)

From the main screen go to the **Update** tab. In the **Status** screen click on **Check Updates**.

[![](/images/vma_check_updates.png "Check Updates")](/images/vma_check_updates.png)

After a few seconds a message will appear showing the new update available.

[![](/images/vma_available_updates_screen.png "Available Updates")](/images/vma_available_updates_screen.png)

Click on **Install Updates** and after asking for confirmation the update process will start.

[![](/images/vma_installing_updates.png "Installing Updates")](/images/vma_installing_updates.png)

Once the update process is complete the appliance will ask for a system reboot.

[![](/images/vma_system_reboot.png "System reboot required")](/images/vma_system_reboot.png)

Go to the System tab and perform the reboot. After the reboot is done you can check the new version in the appliance console,

[![](/images/vma_service_console.png "New vMA version")](/images/vma_service_console.png)

And in the `/etc/vma-release` file.

```text
vi-admin@vma:~> cat /etc/vma-release
vMA 5.0.0 BUILD-643553

Copyright (C) 1998-2011 VMware, Inc. All rights reserved.
This product is protected by U.S. and international copyright and
intellectual property laws. VMware products are covered by one or more U.S.
Patent Numbers D617,808, D617,809, D617,810, D617,811, 6,075,938,
6,397,242, 6,496,847, 6,704,925, 6,711,672, 6,725,289, 6,735,601,
6,785,886, 6,789,156, 6,795,966, 6,880,022, 6,883,095, 6,940,980,
6,944,699, 6,961,806, 6,961,941, 6,970,562, 7,017,041, 7,055,032,
7,065,642, 7,069,413, 7,069,435, 7,082,598, 7,089,377, 7,111,086,
7,111,145, 7,117,481, 7,149,310, 7,149,843, 7,155,558, 7,222,221,
7,260,815, 7,260,820, 7,269,683, 7,275,136, 7,277,998, 7,277,999,
7,278,030, 7,281,102, 7,290,253, 7,343,599, 7,356,679, 7,386,720,
7,409,487, 7,412,492, 7,412,702, 7,424,710, 7,428,636, 7,433,951,
7,434,002, 7,447,854, 7,447,903, 7,467,067, 7,475,002, 7,478,173,
7,478,180, 7,478,218, 7,478,388, 7,484,208, 7,487,313, 7,487,314,
7,490,216, 7,500,048, 7,506,122, 7,516,453, 7,529,897, 7,543,301,
7,555,747, 7,565,527, 7,571,471, 7,577,722, 7,581,064, 7,590,715,
7,590,982, 7,594,111, 7,596,594, 7,596,697, 7,599,493, 7,603,704,
7,606,868, 7,620,523, 7,620,766, 7,620,955, 7,624,240, 7,630,493,
7,636,831, 7,657,659, 7,657,937, 7,665,088, 7,672,814, 7,680,919,
7,689,986, 7,693,996, 7,694,101, 7,702,843, 7,707,185, 7,707,285,
7,707,578, 7,716,446, 7,734,045, 7,734,911, 7,734,912, 7,735,136,
7,743,389, 7,761,917, 7,765,543, 7,774,391, 7,779,091, 7,783,779,
7,783,838, 7,793,279, 7,797,748, 7,801,703, 7,802,000, 7,802,248,
7,805,676, 7,814,495, 7,823,145, 7,831,661, 7,831,739, 7,831,761,
7,831,773, 7,840,790, 7,840,839, 7,840,993, 7,844,954, 7,849,098,
7,853,744, 7,853,960, 7,856,419, 7,856,531, 7,856,637, 7,865,663,
7,869,967, 7,886,127, 7,886,148, 7,886,346, 7,890,754, 7,895,437,
7,908,646, 7,912,951, 7,921,197, 7,925,850; patents pending.
VMware, the VMware "boxes" logo and design, Virtual SMP and VMotion are
registered trademarks or trademarks of VMware, Inc. in the United States
and/or other jurisdictions. All other marks and names mentioned herein may
be trademarks of their respective companies.
vi-admin@vma:~>
```

The above procedure use the default VMware repository and your appliance must be able to resolve public DNS addresses and access the internet in order to download de upgrade bits.

Juanma.
