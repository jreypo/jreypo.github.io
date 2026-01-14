---
title: vMotion error, virtual machine refuses to be migrated.
date: 2012-02-07
categories:
- Virtualization
- VMware
tags:
- ESXi
- vim-cmd
- VMware
- VMware DRS
- VMware HA
- vSphere
showComments: true
---

Last night during a patching job in a customer I found the following error for several VMs when I put a host in maintenance mode and DRS tried to evacuate the virtual machines to the other nodes of the cluster.

[![](/images/vmotion_error_drs.png "vMotion error")](/images/vmotion_error_drs.png)

Very strange since as far as I could see the virtual machines were running without errors, I was logged into some of them through SSH, and they also appeared as powered on in vSphere Client.

I decided to go to Tech Support Mode on the ESXi and check the virtual machine power state.

[![](/images/check_vm_state_tsm.png "Check VM state from ESXi TSM")](/images/check_vm_state_tsm.png)

Everything looked exactly as it should be, no error logs, nothing. At this point I decided to restart the ESXi management agents.

[![](/images/restart_esxi_mgt_agents.png "Restart ESXi Management Agents")](/images/restart_esxi_mgt_agents.png)

And it worked, after a few seconds I was able to perform a successful vMotion and the host could be evacuated.

Juanma.
