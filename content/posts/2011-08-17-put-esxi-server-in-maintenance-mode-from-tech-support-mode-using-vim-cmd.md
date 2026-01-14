---
title: Put ESXi server in maintenance mode from Tech Support Mode using vim-cmd
date: 2011-08-17
tags:
- esxi
- sysadmin
- vsphere
showComments: true
---

If you need to put a host in maintenance mode and only have access through **ESXi Tech Support Mode**, either local from DCUI or remote with SSH, in the following quick post I'll show you how to do it using `vim-cmd` command.

Put the host in maintenance mode:

[![](/images/maintenance_mode_enter.png "Enter maintenance mode")](/images/maintenance_mode_enter.png)

Check the state of the host.

[![](/images/maintenance_mode_check.png "Check host state")](/images/maintenance_mode_check.png)

Get the ESXi out of maintenance mode.

[![](/images/maintenance_mode_exit.png "Exit maintenance mode")](/images/maintenance_mode_exit.png)

This procedure works in ESXi 4.x and ESXi 5.

Juanma.
