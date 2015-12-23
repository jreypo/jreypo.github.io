---
layout: post
title: How to create customized configurations for esxtop/resxtop
date: 2011-04-12
type: post
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- ESX
- ESXi
- esxtop
- performance
- resxtop
- sysadmin
- systems administration
- vMA
- VMware
- vSphere
- vSphere CLI
author: juan_manuel_rey
comments: true
---

One of features I like the most of `esxtop/resxtop` is the ability to create customized configurations. This feature gives you the ability to have several pre-defined configuration files to be used under certain circumstances, for example you can have one only to check if there are virtual machines swapping during a heavy workload period.

The post will cover `esxtop` 4.x, the version that comes with vSphere 4.x, however it can be applicable to previous versions as well. First it's important to know that by default esxtop/resxtop stores its configuration in the file `.esxtop4rc`, in the vMA appliance this file is stored in the `vi-admin` user home directory and in the root home directory in the ESX(i) servers.

Now lets create one as an example. I'm using `resxtop` from the vMA so first launch it against the vCenter Server and select one of the ESX(i) hosts.

[![](/images/resxtop1.png "resxtop")]({{site.url}}/images/resxtop1.png)

Now you should see the default `esxtop` screen. We are going to create a configuration that show only some of the memory related counters.

Show the memory screen by pressing `m` and from there press `f` to edit the fields to display.

[![](/images/esxtop_edit_fields2.png "Edit esxtop fields")]({{site.url}}/images/esxtop_edit_fields2.png)

Press the corresponding keys to enable/disable the fields and `a` or `o` keys to toggle its order, then press the space bar to finish. Next `esxtop` returns the memory view and show the newly selected counters.

[![](/images/memory.png "esxtop memory view")]({{site.url}}/images/memory.png)

At this point you can customized the field to display in the other views (CPU, network, etc). When you are done press W to save the config and enter the file name to save the new config in. If you don't enter a file name `esxtop` will save the changes in its default config file, `/home/vi-admin/.esxtop4rc` in the example.

[![](/images/save_custom_file.png "Save esxtop config to file")]({{site.url}}/images/save_custom_file.png)

Exit `esxtop` and run it again but loading the saved config file, instead of the default one, by using` -c <config_file>`.

[![](/images/load_custom_config.png "Load esxtop custom config file")]({{site.url}}/images/load_custom_config.png)

Finally my advise is to read carefully the [Interpreting esxtop 4.1 Statistics](http://communities.vmware.com/docs/DOC-11812 "Interpreting esxtop 4.1 Statistics") document and use the counters that better suits your needs.

Juanma.
