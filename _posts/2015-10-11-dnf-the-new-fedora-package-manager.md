---
title: DNF, the new Fedora package manager
date: 2015-10-11
type: post
classes: wide
published: true
status: publish
categories:
- Linux
- Red Hat
- Sysadmin
tags:
- dnf
- Fedora
- Fedora 22
- Linux
- Photon
- sysadmin
- tdnf
- yum
author: juan_manuel_rey
comments: true
---

**[Fedora 22](https://getfedora.org/)** was released a few months ago and amongst many new features it came with a replacement for `yum` as package manager called `dnf`, or DaNdiFied YUM, oh yes `yum` is still around but it is now considered legacy software. Also DNF will become in the near future the default package manager for RHEL and CentOS so it is for the best that you get familiarized with it sooner than later.

## DNF Commands

The first thing you need to understand about `dnf` is that many commands are basically still the same but there are differences. Package management commands can be executed with almost the same syntax previously used with `yum`.

Search for a package.

```
[jrey@fed22-srv ~]$ sudo dnf search htop
Last metadata expiration check performed 1:25:54 ago on Mon Oct 5 23:47:45 2015.
=================================== N/S Matched: htop ====================================
htop.x86_64 : Interactive process viewer
php-lightopenid.noarch : PHP OpenID library
[jrey@fed22-srv ~]$
```

Install a package.

```
[jrey@fed22-srv ~]$ sudo dnf install htop
```

Remove a package.

```
[jrey@fed22-srv ~]$ sudo dnf remove htop
```

Get information about a package

```
[jrey@fed22-srv ~]$ sudo dnf info htop
Last metadata expiration check performed 1:47:13 ago on Mon Oct 5 23:47:45 2015.
Available Packages
Name : htop
Arch : x86_64
Epoch : 0
Version : 1.0.3
Release : 4.fc22
Size : 91 k
Repo : fedora
Summary : Interactive process viewer
URL : http://hisham.hm/htop/
License : GPL+
Description : htop is an interactive text-mode process viewer for Linux, similar to
 : top(1).

[jrey@fed22-srv ~]$
```

Group and repository management commands are still the same as well.

```
[jrey@fed22-srv ~]$ sudo dnf repolist
```

Querying the available repositories for a specific command.

```
[jrey@fed22-srv ~]$ sudo dnf repoquery --whatprovides htop
Last metadata expiration check performed 1:54:52 ago on Mon Oct 5 23:47:45 2015.
htop-0:1.0.3-4.fc22.x86_64
[jrey@fed22-srv ~]$
```

`dnf` comes with some powerful capabilities like history query.

```
[jrey@fed22-srv ~]$ sudo dnf history list
Last metadata expiration check performed 11 days, 19:14:54 ago on Wed Oct 7 02:56:21 2015.
ID | Command line             | Date a           | Action  | Altere
-------------------------------------------------------------------------------
 9 | history undo 8           | 2015-10-06 01:53 | Install | 1
 8 | erase htop               | 2015-10-06 01:28 | Erase   | 1
 7 | install htop -y          | 2015-10-06 01:28 | Install | 1
 6 | remove htop              | 2015-10-06 01:14 | Erase   | 1
 5 | install htop             | 2015-10-06 01:14 | Install | 1
 4 | install make gcc kernel- | 2015-09-30 16:21 | Install | 9
 3 | update                   | 2015-09-30 15:43 | I, U    | 112
 2 | update                   | 2015-09-16 11:45 | I, O, U | 297
 1 |                          | 2015-09-16 10:59 | Install | 658 EE
[jrey@fed22-srv ~]$
```

This can be specially helpful if you need to rollback a change, like clean up dependencies after uninstalling a package or reinstall a package.

```
[jrey@fed22-srv ~]$ sudo history undo 8
```

You can also look for duplicated within the installed ones.

```
[jrey@fed22-srv ~]$ sudo dnf repoquery --duplicated
Last metadata expiration check performed 0:30:42 ago on Tue Oct 6 02:48:41 2015.
kernel-core-0:4.0.4-301.fc22.x86_64
kernel-core-0:4.1.6-201.fc22.x86_64
kernel-core-0:4.1.7-200.fc22.x86_64
kernel-modules-0:4.0.4-301.fc22.x86_64
kernel-modules-0:4.1.6-201.fc22.x86_64
kernel-modules-0:4.1.7-200.fc22.x86_64
[jrey@fed22-srv ~]$
```

Retrieve all available packages providing a specific software of capability.

```
[jrey@fed22-srv ~]$ sudo dnf repoquery --whatprovides curl
Last metadata expiration check performed 0:38:00 ago on Tue Oct 6 02:48:41 2015.
curl-0:7.40.0-3.fc22.x86_64
curl-0:7.40.0-7.fc22.x86_64
[jrey@fed22-srv ~]$
```

This is a very basic introduction to `dnf` capabilities but hopefully you have been able to get how it works. My advice is to review [DNF documentation](http://dnf.readthedocs.org/en/latest/index.html) for all the details.

## The Photon Connection

[**VMware Photon**](https://vmware.github.io/photon/) comes with `tdnf` (Tiny DNF); this is a development by VMware that comes with compatible repository and package management capabilities. Not every `dnf` command is available but the basic ones are there.

Package installation and updates.

[![](/assets/images/screen-shot-2015-10-11-at-19-41-00.png)]({{site.url}}/assets/images/screen-shot-2015-10-11-at-19-41-00.png)

Repository management.

[![](/assets/images/screen-shot-2015-10-11-at-18-54-47.png)]({{site.url}}/assets/images/screen-shot-2015-10-11-at-18-54-47.png)

In the future if I find the time I'll write a new post with some advanced examples of `dnf` commands. Comments are welcome.

Juanma.
