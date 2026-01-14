---
title: Monitor ESX 4.x to ESXi 5.0 migration process
date: 2011-11-04
categories:
- Virtualization
- VMware
tags:
- ESX4
- ESXi Shell
- ESXi5
- vmkernel
showComments: true
---

During the migration of an ESX 4.x to ESXi 5.0 the whole process can be monitored directly from the console of the server.

Once the process has started press **Alt-F1** to access the Console. Login with root and blank password.

[![](/images/console_login.png "ESXi Console Login")](/images/console_login.png)

From here you can go to the `/var/log` folder and using the `tail` command to monitor ESXi log files.

Also by pressing `Alt-F12` you will see the `vmkernel` log, this log will show the upgrade process in real time. Once the log reaches the point in the screenshot the upgrade will be complete.

[![](/images/install_finished.png "Installation finished")](/images/install_finished.png)

At this point and before restarting the host if you go back again to the ESXi console you can review the ESXi install log file, called `esxi_install.log` which in fact is a symlink to the file `weasel.log`.

[![](/images/esxi_install_log.png "ESXi install log")](/images/esxi_install_log.png)

In this log file you can observe the whole migration process, I strongly recommend to lose a few minutes on this since you will learn a lot of under the hood info about the ESXi installation process.

Finally and only as a curiosity after the reboot if you login into the ESXi Shell a message indicating that the system has been migrated to ESXi 5.0 will be displayed before the prompt.

[![](/images/esxishell_first_login.png "ESXi Shell first login")](/images/esxishell_first_login.png)

Juanma.
