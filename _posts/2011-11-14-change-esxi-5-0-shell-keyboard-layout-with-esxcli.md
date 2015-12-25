---
layout: post
title: Change ESXi 5.0 Shell keyboard layout with esxcli
date: 2011-11-14
type: post
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- esxcli
- ESXi Shell
- ESXi5
- sysadmin
- systems administration
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

If you have to login into the ESXi 5.0 Shell and the keyboard layout is not the one you are used to this post will show how to quickly change it.

As always in vSphere 5 we are going to use `esxcli` command to get the job done.

#### Get current keyboard layout

[![](/images/esxi_key_layout.png "Get current layout")]({{site.url}}/images/esxi_key_layout.png)

As you can see we are using `system settings keyboard layout` namespaces and the command `get`. The other available commands are `list` and `set`.

#### List available layouts

[![](/images/esx_avail_layouts.png "List available layouts")]({{site.url}}/images/esx_avail_layouts.png)

#### Change keyboard layout

The syntax for the command can be retrieved by appending `–help` to the command.

[![](/images/esxicli_key_help.png)]({{site.url}}/images/esxicli_key_help.png)

Now change the layout to `US Default`.

[![](/images/esxcli_change_key_layout.png "Change ESXi keyboard layout")]({{site.url}}/images/esxcli_change_key_layout.png)

Keep in mind that this will change the layout permanently, as it can be seen in the command help the layout can also be changed only for the current boot and it will be reset to its original value during next reboot of the host.

[![](/images/change_esxi_key_layout_nopersist.png)]({{site.url}}/images/change_esxi_key_layout_nopersist.png)

With the `—no-persist` option the host will report its original layout.

Juanma.
