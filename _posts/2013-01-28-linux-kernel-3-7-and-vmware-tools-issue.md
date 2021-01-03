---
title: Linux Kernel 3.7 and VMware Tools issue
date: 2013-01-28
type: post
classes: wide
published: true
status: publish
categories:
- Linux
- Sysadmin
- Virtualization
- VMware
tags:
- Fedora
- Kernel 3.7
- Linux
- sysadmin
- Virtualization
- VMware
- VMware Fusion
- VMware Tools
author: juan_manuel_rey
comments: true
---

I got aware of this issue last week after installing a Fedora 18 virtual machine on Fusion 5. The installation of the Tools went as expected but when the install process launched the `vmware-tools-config,pl` script I got the typical error of not being able to find the Linux Kernel headers.

```
Searching for a valid kernel header path...
The path "" is not a valid path to the 3.7.2-204.fc18.x86_64 kernel headers.
Would you like to change it? [yes]
```

I installed the `kernel-headers` and `kernel-devel` packages with `yum`.

```
[root@fed18 ~]# yum install kernel-headers kernel-devel
```

Fired up again the configuration script and got the same error. The problem is that since kernel 3.7 all the kernel header files have been relocated to a new path and because of that the script is not able to find them. To solve it just create a symlink of the *version.h* file from the new location to the old one.

```
[root@fed18 src]# ln -s /usr/src/kernels/3.7.2-204.fc18.x86_64/include/generated/uapi/linux/version.h /lib/modules/3.7.2-204.fc18.x86_64/build/include/linux/
```

With the problem fixed I launched the config script again and the tools finally got configured without problems.

```
[root@fed18 ~]# vmware-config-tools.pl
Initializing...

Making sure services for VMware Tools are stopped.
Stopping Thinprint services in the virtual machine:
 Stopping Virtual Printing daemon: done
Stopping vmware-tools (via systemctl): [ OK ]

The VMware FileSystem Sync Driver (vmsync) allows external third-party backup
software that is integrated with vSphere to create backups of the virtual
machine. Do you wish to enable this feature? [no]

Before you can compile modules, you need to have the following installed...
make
gcc
kernel headers of the running kernel

Searching for GCC...
Detected GCC binary at "/bin/gcc".
The path "/bin/gcc" appears to be a valid path to the gcc binary.
Would you like to change it? [no]

Searching for a valid kernel header path...
Detected the kernel headers at
"/lib/modules/3.7.2-204.fc18.x86_64/build/include".
The path "/lib/modules/3.7.2-204.fc18.x86_64/build/include" appears to be a
valid path to the 3.7.2-204.fc18.x86_64 kernel headers.
Would you like to change it? [no]
```

Juanma.
