---
layout: post
title: Howto get VxVM diskgroup version
date: 2010-05-21 08:54:59.000000000 +02:00
type: post
published: true
status: publish
categories:
- HP-UX
- Sysadmin
tags:
- HP-UX
- VxVM
author: juan_manuel_rey
---

If you need to determine the version of a Veritas `diskgroup` it can be done by two ways:

-   `vxdg` command:

Execute `vxdg list <diskgroup>` and look for the version field in the output.

{% highlight text %}
root@vmnode1:~# vxdg list dg_sap
Group:     dg_sap
dgid:      1273503890.14.vmnode1
import-id: 1024.10
flags:     cds
version:   140 <--- VERSION!
alignment: 8192 (bytes)
local-activation: read-write
ssb:            on
detach-policy: global
dg-fail-policy: dgdisable
copies:    nconfig=default nlog=default
config:    seqno=0.1076 permlen=24072 free=24068 templen=2 loglen=3648
config disk disk27 copy 1 len=24072 state=clean online
config disk disk28 copy 1 len=24072 state=clean online
log disk disk27 copy 1 len=3648
log disk disk28 copy 1 len=3648
root@vmnode1:~#
{% endhighlight %}

-   `vxprint` command:

Run `vxprint -l <diskgroup>` and again look for the versión field as shown in the example.

{% highlight text %}
root@vmnode1:~# vxprint -l dg_sap
Disk group: dg_sap

Group:    dg_sap
info:     dgid=1273503890.14.vmnode1
version:  140 <--- VERSION!
alignment: 8192 (bytes)
activation: read-write
detach-policy: global
dg-fail-policy: dgdisable
copies:   nconfig=default nlog=default
devices:  max=32767 cur=1
minors:   >= 4000
cds=on

root@vmnode1:~#
{% endhighlight %}

And as Nelson Muntz like to say... smell you later ;-)

Juanma.