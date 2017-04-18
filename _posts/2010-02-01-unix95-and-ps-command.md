---
layout: post
title: UNIX95 and ps command
date: 2010-02-01
type: post
published: true
status: publish
categories:
- HP-UX
- Sysadmin
tags:
- commands
- HP-UX
- sysadmin
- systems administration
- Unix
author: juan_manuel_rey
comments: true
---

**UNIX95** is a Unix standard defined in the [Single UNIX Specification](http://www.unix.org/what_is_unix/single_unix_specification.html). HP-UX 11i is registered as UNIX95 compliant in its v1 and v2 versions and as UNIX03 (a more modern version) the v3.

This standard affect commands and syscalls in various ways and it is not set by default. To "activate" this mode the UNIX95 environment variable must be declared, it is not recommendable to set the variable through root's `.profile` or `/etc/profile`. The best way to use the features provided by the UNIX95 standard is to temporally set the variable just before execute a command in the shell.

Currently I only use this mode with the ps command. This command has some options that can only be used if the UNIX95 variable is declared. I have a couple of alias in my profile to take advantage of this options.

-   With this command you can get a sort of equivalent of the Solaris `ptree` command.

```
root@asoka:/# alias ptree
alias ptree='UNIX95=1 ps -eHf'
```

-   Get the top 15 CPU consuming processes:

```
root@asoka:/# alias pcpu
alias pcpu='UNIX95=1 ps -ef -o pcpu,user,pid,args | /bin/sort -u -r | sed -e '\''s/\.[0-9][0-9]/&\%/g'\'' | sed -n 1,15p'
root@asoka:/# pcpu
 0.83% root     10520 /opt/wbem/lbin/cimprovagt 0 5 10 root SFMProviderModule
 0.51% hpsmh    10434 /opt/hpws/apache/bin/httpd -k start -DSSL -f /opt/hpsmh/conf/smhpd.conf
 0.45% root        53 vxfsd
 0.42% cimsrvr  10518 cimservermain --executor-socket 5
 0.16% root      1904 vxsvc -r /opt/VRTSob/config/Registry -e
 0.13% root       552 /usr/sbin/utmpd
 0.12% root      1468 /opt/dce/sbin/rpcd
 0.11% root        39 schedcpu
 0.10% root     10523 /opt/wbem/lbin/cimprovagt 0 5 10 root HPUXFCIndicationProviderModule
 0.10% root      1707 /usr/sbin/swagentd -r
 0.07% root     26676 sshd: root@pts/1
 0.06% root     10525 /opt/wbem/lbin/cimprovagt 0 5 10 root HPUXLANIndicationProviderModule
 0.06% root      2296 /usr/sbin/stm/uut/bin/tools/monitor/WbemWrapperMonitor
 0.06% root      1028 /usr/sbin/automountd
 0.06% root        21 ksyncer_daemon
root@asoka:/#
```

There are more options for ps and for other commands, just investigate a bit in the man pages.

Juanma.
