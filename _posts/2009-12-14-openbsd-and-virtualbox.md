---
layout: post
title: OpenBSD and VirtualBox
date: 2009-12-14 16:06:38.000000000 +01:00
type: post
published: true
status: publish
categories:
- BSD
- Virtualization
tags:
- OpenBSD
- VirtualBox
author: juan_manuel_rey
---

I've been using **OpenBSD** since the 3.0 version as desktop, in my home servers and even in production systems, some time ago I decided to virtualize my OpenBSD infrastructure first with VMware Server and later with VirtualBox (both of them with Linux as the host system) which is much more powerfull; but when I started to use VirtualBox a year or so ago I discovered that installing OpenBSD in VirtualBox can be a pain in the ass.

The installation went smoothly until I got the following error:

{% highlight text %}
uid 0 on /: file system full
/: write failed, file system is full
Segmentation fault
{% endhighlight %}

Sometimes the installation started despite of the error but it did not finish correctly, others  the installation process hung up and had to forcibly shutdown the VM or simply the installation aborted. I tried several times and got some weird errors and the previous error not always at the same step.

I did a small research and found a couple of solutions for this issue. The first solution was to enable VT-x/AMD-V virtualization extensions, but my processor didn't have those extensions so this solution didn't solve my problem.

The second workaround finally allowed me to install OpenBSD without errors. The virtual machine has to be started from the command line with the `-norawr0` option:

{% highlight text %}
jmr@wopr:~# VirtualBox -startvm <uid_or_name_of_the_vm> -norawr0

jmr@wopr:~# VBoxSDL -norawr0 -vm <uid_or_name_of_the_vm>
{% endhighlight %}

One final note, with OpenBSD 4.6 I didn't get the error, even with a processor without the VT-x extensions.

Juanma.
