---
layout: post
title: Installing the HP Lefthand CMC in Linux
date: 2010-07-27
type: post
published: true
status: publish
categories:
- Linux
- Storage
tags:
- HP Lefthand
- Lefthand CMC
- Linux
- Storage
author: juan_manuel_rey
comments: true
---

May be some of you are not aware of this but the HP Lefthand Central Management Console application is available not only for Windows but for Linux and HP-UX also. The application is included on the SAN/iQ Management Software DVD that can be downloaded from [here](http://www.hp.com/go/P4000downloads).

Burn the iso  or mount it in your Linux system. Navigate trough the iso to `GUI/Linux/Disk1/InstData`, there you will find two files and a directory named `VM`. Get into the directory and will find the installer `CMC_Installer.bin`.

Launch the installer passing it the full path to the installer properties file, in this case the file `MediaId.properties` that can be found on `GUI/Linux/Disk1/InstData`.

```
root@wopr:/mnt/iso/GUI/Linux/Disk1/InstData/VM# ./CMC_Installer.bin -f /mnt/iso/Linux/Disk1/InstData/MediaId.properties
```

The CMC will be installed in `/opt/LeftHandNetworks/UI`. Once the installation is finished launch the CMC from the shell or create a launcher on your Gnome/KDE desktop and voilà you can now control your Lefthand Storage systems from your favorite Linux distro.

[![](/images/cmc_ubuntu.png "CMC_Ubuntu")]({{site.url}}/images/cmc_ubuntu.png)

Juanma.
