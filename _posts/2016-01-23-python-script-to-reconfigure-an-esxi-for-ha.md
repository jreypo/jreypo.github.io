---
layout: post
title: Python script to reconfigure an ESXi for HA
date: 2016-01-23
type: post
published: true
status: publish
categories:
- Virtualization
- VMware
- Sysadmin
- DevOps
tags:
- devops
- sysadmin
- VIO
- Python
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

If you have following my blog for some time you may remember a [post from 2011]({% post_url 2011-02-21-reconfigure-a-esxi-for-ha-using-vsphere-powercli %}) where I wrote about the procedure perform a **Reconfigure for HA** operation in an ESXi host from PowerCLI command line, I even published a small and dirty script to do it in one command. I've been thinking for some time to convert to Python that and many other vSphere scripts I have in PowerCLI and Perl and finally I decide to do it and this is the first result.

The usage of the script is very simple and follow the guidelines used in other pyVmomi scripts. you need to provide the vCenter Server, user with administrative rights and of course the ESXi host.

```
./reconfigure_host_for_ha.py -s vcsa-01.jreypo.io -u administrator@starlabs.local -e vsan-esx-01.jreypo.io
```

I've found an issue with the verification of vCenter SSL certificate and `SmartConnect()`, described in [this pyVmomi issue](https://github.com/vmware/pyvmomi/issues/235) and in the same issue I found the workaround provided by another user.

Grab the code from the below [gist](https://gist.github.com/jreypo/1fd33accaee46c31c1b6).

{% gist jreypo/1fd33accaee46c31c1b6 %}

I've created a new repository on Github called [`pvymomi-scripts`](https://github.com/jreypo/pyvmomi-scripts) where I'll be publishing more of my old scripts as soon as I code them in Python. Also, at least for this one, I have opened a pull request to [pyVmomi Community Samples]() repository to get it included.

-- Juanma
