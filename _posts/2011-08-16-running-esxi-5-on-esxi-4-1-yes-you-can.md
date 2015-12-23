---
layout: post
title: Running ESXi 5 on ESXi 4.1? Yes you can!
date: 2011-08-16
type: post
published: true
status: publish
categories:
- Virtualization
- VMware
tags:
- ESXi
- ESXi4
- ESXi5
- homelab
- Virtualization
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

If you are wondering if you can run your vSphere 5 lab nested on ESXi 4.1, the answer is yes.

I used **Eric Gray's** ([@eric\_gray](http://twitter.com/#!/eric_gray)) procedure [VMware ESX 4 can even virtualize itself](http://www.vcritical.com/2009/05/vmware-esx-4-can-even-virtualize-itself/) to create the VMs. For the guest OS type I tried Red Hat Enterprise Linux 5 (64-bit) and Red Hat Enterprise Linux 6 (64-bit) and both worked without a glitch.

Here they are running on top of my whitebox, which is running ESXi 4.1 Update 1, the left one (esxi5) is created as RHEL6 and the right one (esxi5-02) RHEL5.

[![](/images/esxi5.png "ESXi5")]({{site.url}}/images/esxi5.png)

I added also the `monitor_control.restrict_backdoor` option but have not try yet to run nested VMs. I'll do later and will update the post with the results.

Juanma
