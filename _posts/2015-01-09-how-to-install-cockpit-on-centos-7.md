---
title: How to install Cockpit on CentOS 7
date: 2015-01-09
type: post
classes: wide
published: true
status: publish
categories:
- Linux
- Red Hat
- Sysadmin
tags:
- Atomic
- CentOS
- CentOS 7
- Cockpit
- devops
- Linux
- Red Hat
- sysadmin
- systems administration
author: juan_manuel_rey
comments: true
---

Being used to have [**Cockpit**](http://cockpit-project.org/) in my Fedora 21 Server VMs I decided that having it also on my [**CentOS**](http://centos.org/) machines would be awesome, unfortunately I quickly found that Cockpit was not available in CentOS repositories. Of course I knew that Cockpit comes installed and enabled by default in CentOS 7 Atomic host image so I figured out that those packages had to be hidden in some Atomic related repo.

After looking a bit I finally found in GitHub the [sig-atomic-buildscripts](https://github.com/CentOS/sig-atomic-buildscripts) repository that belongs to CentOS Project. This repository contains several scripts and files intended to build your own CentOS Atomic host including `virt7-testing.repo`, the Yum repository file needed for Cockpit.

Clone the GutHub repository.

```text
git clone https://github.com/baude/sig-atomic-buildscripts
```

Copy `virt7-testing.repo` file to `/etc/yum.repos.d` and install Cockpit.

```text
yum install cockpit
```

Enable Cockpit service.

```text
[root@webtest ~]# systemctl enable cockpit.socket
ln -s '/usr/lib/systemd/system/cockpit.socket' '/etc/systemd/system/sockets.target.wants/cockpit.socket'
[root@webtest ~]#
```

Add Cockpit to the list of trusted services in FirewallD.

```text
[root@webtest ~]# firewall-cmd --permanent --zone=public --add-service=cockpit
success
[root@webtest ~]#
[root@webtest ~]# firewall-cmd --reload
success
[root@webtest ~]#
[root@webtest ~]# firewall-cmd --list-services
cockpit dhcpv6-client ssh
[root@webtest ~]#
```

Start Cockpit socket.

```text
systemctl start cockpit.socket
```

Do no try to access Cockpit yet, there is an [issue](https://github.com/cockpit-project/cockpit/issues/1581) about running Cockpit on stock CentOS/RHEL 7. To be able to start it we need first to modify the service file to disable SSL. Edit file `/usr/lib/systemd/system/cockpit.service` and modify `ExecStart` line to look like this.

```ini
ExecStart=/usr/libexec/cockpit-ws --no-tls
```

I know this procedure will invalidate Cockpit for a production environment in RHEL7 at least for now but this is for my lab environment and I can live with it.

Reload `systemd`.

```text
systemctl daemon-reload
```

Restart Cockpit.

```text
systemctl restart cockpit
```

Access Cockpit web interface, login as root and have fun :-)

[![](/assets/images/screen-shot-2015-01-09-at-01-57-51.png)]({{site.url}}/assets/images/screen-shot-2015-01-09-at-01-57-51.png)

Juanma.
