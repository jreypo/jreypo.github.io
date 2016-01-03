---
layout: post
title: Managing your Fedora Server with Cockpit
date: 2015-01-08
type: post
published: true
status: publish
categories:
- DevOps
- Linux
- Red Hat
- Sysadmin
tags:
- Cockpit
- devops
- Fedora
- Fedora 21
- Fedora Server
- Linux
- Red Hat
- sysadmin
- systems management
author: juan_manuel_rey
comments: true
---

[Cockpit](http://cockpit-project.org) is a new web based server manager to administer Linux server, it will provide the system administrators with a user friendly interface to manage their Linux servers, it includes multi-server managing capacity and more importantly it will create no interference or disconnection between the tasks done from the web and from the command line. This last feature is specially useful

By default Cockpit, stable version, comes installed and enabled in [Fedora 21 Server](https://getfedora.org/en/server/). It also can be found in CentOS/RHEL 7 Atomic, Fedora 21 Atomic and Fedora 21 Cloud, and there are plans in the near future to support [Arch Linux](https://www.archlinux.org/).

Lets review now some of the features of Cockpit, as said before multiple servers can be managed from the same Cockpit instance.

[![](%7B%7B%20site.baseurl%20%7D%7D/assets/screen-shot-2014-12-31-at-19-26-00.png?w=580)](https://jreypo.files.wordpress.com/2014/12/screen-shot-2014-12-31-at-19-26-00.png)

Once you access one of managed nodes it will present general overview of the server with real-time charts of CPU, Memory, Disk I/O and Network Traffic.

[![](%7B%7B%20site.baseurl%20%7D%7D/assets/screen-shot-2014-12-31-at-19-37-53.png?w=580)](https://jreypo.files.wordpress.com/2014/12/screen-shot-2014-12-31-at-19-37-53.png)

On the left pane there are a series of actionable items that will give you access to the different subsystems of the node like Networking, Storage, User Accounts and even the status of the Docker containers running on the server, if the Docker service has been enabled.

System services view.

[![](%7B%7B%20site.baseurl%20%7D%7D/assets/screen-shot-2014-12-31-at-19-52-53.png?w=580)](https://jreypo.files.wordpress.com/2015/01/screen-shot-2014-12-31-at-19-52-53.png)

When a process is selected Cockpit will display its details.

[![](%7B%7B%20site.baseurl%20%7D%7D/assets/screen-shot-2015-01-08-at-12-11-27.png?w=580)](https://jreypo.files.wordpress.com/2015/01/screen-shot-2015-01-08-at-12-11-27.png)

Networking area displays traffic for the selected interface, the journal of the networking system and even allows you to create a new bond interface, a new bridge or add a new VLAN tag to the interface.

[![](%7B%7B%20site.baseurl%20%7D%7D/assets/screen-shot-2014-12-31-at-19-53-18.png?w=580)](https://jreypo.files.wordpress.com/2015/01/screen-shot-2014-12-31-at-19-53-18.png)

The Storage view will display similar info for the disks, and will display detailed information for each of them, review the LVM configuration of the server and perform different storage related operations.

[![](%7B%7B%20site.baseurl%20%7D%7D/assets/screen-shot-2014-12-31-at-19-53-45.png?w=580)](https://jreypo.files.wordpress.com/2015/01/screen-shot-2014-12-31-at-19-53-45.png)

Journal view lets you review `systemd` journal. You can go back seven days into the log and filter on the type of messages.

[![](%7B%7B%20site.baseurl%20%7D%7D/assets/screen-shot-2014-12-31-at-19-54-29.png?w=580)](https://jreypo.files.wordpress.com/2015/01/screen-shot-2014-12-31-at-19-54-29.png)

After using Cockpit for some time in my lab I can say that I genuinely love it, the interface is pretty fast, it uses `systemd` for everything and it does not interface with my console-based admin habits, on the contrary is a great complement to them.

Juanma.
