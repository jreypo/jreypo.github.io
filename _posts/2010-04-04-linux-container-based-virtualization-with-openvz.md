---
title: Linux container-based virtualization with OpenVZ
date: 2010-04-04
type: post
classes: wide
published: true
status: publish
categories:
- Linux
- Virtualization
tags:
- containers
- Linux
- OpenVZ
- Parallels
author: juan_manuel_rey
comments: true
---

Long time since my last post. I've been on holidays! :-D

But don't worry my dear readers, I did not fall into laziness and the few times I wasn't playing with my son I've been playing in my homelab with other virtualization technologies, storage appliances, my ESXi servers... what can I say I'm a Geek. One of the most interesting technologies I've been playing with is **OpenVZ**.

[OpenVZ](http://wiki.openvz.org/Main_Page "OpenVZ Wiki")is a container-based operating system-level virtualization technology for Linux, if you have ever worked with Solaris 10 Zones this is very similar. The OpenVZ project is supported by Parallels which also have based their commercial solution Virtuozzo on OpenVZ.

OpenVZ (and other container-based technologies) differs with other technologies like VMware or HPVM in that the last ones virtualize the entire machine with its own OS, disks, ram, etc. OpenVZ on the contrary use only one Linux kernel and create multiple isolated instances. of course there are pros and cons, just to name a couple:

-   VMware, HPVM and other true hypervisors are more flexible since many different operative systems can be run on top of them, OpenVZ on the contrary can only run Linux instances.
-   OpenVZ since it is not a real hypervisor does not have their overhead so it is very fast.

### Glossary of terms

-   Host node, CTo, VEo: The host where the containers run.
-   VPS, VE: The containers themselves. One Host node can run multiple VPS and each VPS can run a different Linux distribution such as Gentoo, Ubuntu, CentOS, etc, but every VPS operate under the same Linux kernel.
-   CTID: ConTainer's IDentifier. A unique number that every VPS has and used to manage it.
-   Beancounters: The beancounters, also known as UBC Parameter Units, are nothing but a set of limits defined by the system administrator. The beancounters assure that no VPS can abuse the resources of the Host node. The whole list of Beancounter is described in detail in the [OpenVZ wiki](http://wiki.openvz.org/UBC_parameter_units "OpenVZ UBC Parameter Units").
-   VPS templates: The templates are the images used to create new containers.

### OpenVZ directory structure

-   `/vz` - The default main directory.
-   `/vz/private` - Where the VPS are stored.
-   `/vz/template/cache` - In this path are stored the templates for the
    different Linux distributions.
-   `/etc/vz` - Configuration directory for OpenVZ.
-   `/etc/vz/vz.conf` - OpenVZ configuration file.

### Resource management

The amount of resources from the Host node available for the Virtual Environments can be managed through four different ways.

-   Two-Level Disk Quota: The administrator of the Host node can set-up disk quotas for each  container.
-   Fair CPU scheduler: It is a two-level scheduler with a first level scheduler deciding which container is given the CPU time slice and on the second level the standard Linux scheduler decides which process to run in that container.
-   I/O scheduler: Very similar to the CPU scheduler, is also a two-level scheduler. Priorities are assigned to each container and the I/O scheduler distributes the available bandwidth according to those priorities.
-   User Beancounters.

I'm now in the process of set-up an OpenVZ test server in my homelab so I will try to cover some of its features more in depth in a future post.

Juanma.
