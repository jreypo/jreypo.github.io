---
layout: post
title: HPVM memory management
date: 2010-06-09
type: post
published: true
status: publish
categories:
- HP
- HP-UX
- Itanium
- Virtualization
tags:
- Autmatic Memory Reallocation
- HP-UX
- HPVM
- HPVM 4.2
- HPVM Dynamic Memory
- Integrity VMs
author: juan_manuel_rey
comments: true
---

Like other virtualization software, HP Integrity Virtual Machines comes with several memory management capabilities. In this new post about HPVM I will try to explain what are these capabilities, their purpose and how to configure and use them.

### Dynamic memory

Dynamic memory is an HPVM feature that allow you to resize the amount of memory of a guest without rebooting it. The HPVM manual mention an example in which dynamic memory is applicable.

>...this feature allows a guest that is a Serviceguard node to be used
as a standby server for multiple Serviceguard packages. When a package
fails over to the guest, the guest memory can be changed to suit the
requirements of the package before, during, and after the failover
process.

Dynamic memory is only available on HP-UX guests with the guest management software installed.

Lets see how to enable an configure dynamic memory.

First thing to do is to enable dynamic memory.

{% highlight text %}
root@hinata:~ # hpvmmodify -P batman -x ram_dyn_type=driver
{% endhighlight %}

There are three possible values for the `ram_dyn_type` option:

1.  None: Self explanatory.
2.  Any: In the next boot of the guest it will check if dynamic memory is enabled and if the driver is loaded. If the dynamic memory driver is in place the option will change its value to `driver`.
3.  Driver: When the `ram_dyn_type` is set to `driver` it means that every dynamic memory control and range is functional.

Specify the minimum amount of RAM to be allocated to the guest, the default unit is MB but GB can also be used.

{% highlight text %}
root@hinata:~ # hpvmmodify -P batman -x ram_dyn_min=1024
{% endhighlight %}

Next set the maximum memory.

{% highlight text %}
root@hinata:~ # hpvmmodify -P batman -x ram_dyn_max=4G
{% endhighlight %}

Set the amount of memory to be allocated when the guests starts, this value must be greater than the minimum one.

{% highlight text %}
root@hinata:~ # hpvmmodify -P batman -x ram_dyn_target_start=2048
{% endhighlight %}

Check the status of the guest to see the newly configured options.

{% highlight text %}
root@hinata:~ # hpvmstatus -r -P batman
[Virtual Machine entitlements]
 Percent       Cumulative
#VCPUs Entitlement Maximum   Usage            Usage
====== =========== ======= ======= ================
 6       10.0%  100.0%    0.0%                0

[Virtual CPU details]
vCPU Cumulative       Guest   Host    Cycles   Sampling
ID   Usage            percent percent achieved Interval
==== ================ ======= ======= ======== ===========
 0                0    0.0%    0.0%     0MHz   0 seconds
 1                0    0.0%    0.0%     0MHz   0 seconds
 2                0    0.0%    0.0%     0MHz   0 seconds
 3                0    0.0%    0.0%     0MHz   0 seconds
 4                0    0.0%    0.0%     0MHz   0 seconds
 5                0    0.0%    0.0%     0MHz   0 seconds

[Virtual Machine Memory Entitlement]
DynMem  Memory   DynMem  DynMem DynMem  Comfort Total    Free   Avail    Mem    AMR     AMR
 Min   Entitle   Max    Target Current   Min   Memory  Memory  Memory  Press  Chunk   State
======= ======= ======= ======= ======= ======= ======= ======= ======= ===== ======= ========
 1024MB     0MB     4GB  4096MB  4096MB     0MB     4GB     0MB     0MB   0       0MB DISABLED
{% endhighlight %}

Once dynamic memory is properly configured, from the VM host, the memory of a guest can be manually resized to a value between the `ram_dyn_min` and `ram_dyn_max` parameters in increments of the default chunk size, which is 64MB.

{% highlight text %}
root@hinata:~ # hpvmmodify -P batman -x ram_target=3136
{% endhighlight %}

There is one final option named `dynamic_memory_control`, with this option the system administration can allow the root user of the guest to change dynamic memory options, from the guest side, while it is running. The `dynamic_memory_control` option is incompatible with automatic memory reallocation.

Just to show a small example from the guest side, to view the dynamic memory configuration:

{% highlight text %}
root@batman:~# hpvmmgmt -V -l ram
[Dynamic Memory Information]
=======================================
Type                    : driver
Minimum memory          : 1024 MB
Target memory           : 4090 MB
Maximum memory          : 4096 MB
Current memory          : 4090 MB
Comfortable minimum     : 1850 MB
Boot memory             : 4090 MB
Free memory             : 2210 MB
Available memory        : 505 MB
Memory pressure         : 0
Memory chunksize        : 65536 KB
Driver Mode(s)          : STARTED ENABLED

root@batman:~#
{% endhighlight %}

### Automatic memory reallocation

The new HPVM 4.2 version from March expands dynamic memory with an interesting feature called **Automatic Memory Reallocation**. This new feature provides the possibility of automated changes in the amount of memory used by a guest based on load conditions.

Automatic memory reallocation is only supported on HP-UX guests with dynamic memory enabled and with the guest management software installed.

Automatic memory reallocation can be configured through two ways:

1.  System-wide values.
2.  On a per-VM basis.

Each way doesn't exclude the other one, you can set the system-wide parameters for every VM and later customize some of the virtual machines adjusting their parameters to any additional requirement.

Automatic memory reallocation is enabled by default on the VM host. Open the file `/etc/rc.config.d/hpvmconf` and check that `HPVMAMRENABLE=0` is not set to verify that automatic memory reallocation is enabled. The process `hpmvmamrd`, the automatic memory reallocation daemon, can also be check with a simple `ps`.

[![](/images/hpvmamrd.jpg "hpvmamrd")]({{site.url}}/images/hpvmamrd.jpg)

In the same file two system-wide tunables can be configured.

1.  `HPVMCHUNKSIZE`
2.  `HPVMAMRWAITTIME`

The first parameter determine the number of megabytes by the guest will attempt to grow if there is memory pressure. If the parameter is not set the default value will be 256MB. The best practice for this parameter is to be a multiple of the dynamic memory chunk size.

The second one set the maximum number of seconds that any VM startup process will wait for memory before reporting a failure due to insufficient memory. The default value is 60 seconds and the maximum configurable 600 seconds.

With the above parameter set to its defaults or customized the next step is to enable automatic memory reallocation in the virtual machines. The `amr` feature is DISABLED by default on the VMs. To enable use the `amr_enable` option.

{% highlight text %}
root@hinata:~ # hpvmmodify -P batman -x amr_enable=1
{% endhighlight %}

Now set the memory entitlement for the virtual machine. The entitlement is the minimum amount of RAM guaranteed to the virtual machine.

{% highlight text %}
root@hinata:~ # hpvmmodify -P batman -x ram_dyn_entitlement=1500
{% endhighlight %}

Take into account that if `amr` is not enabled the entitlement could be set but it will not work and any VM without the entitlement parameter set will be ignored by automatic memory reallocation.

The entitlement value can be modified online by the system administrator at any time, but there are some rules that apply:

1.  If there is not enough memory to grow the VM memory to the specified
    entitlement the operation will fail.
2.  The memory of virtual machine can not be grown beyond its
    maximum memory.
3.  The virtual machine memory always have to be set to a value between
    `ram_dyn_max` and `ram_dyn_min` parameters, no more no less.

When the memory of a guest is resized by default the `HPVMCHUNKSIZE` value is used but a per-VM chunk size can also be set. To do so use the `amr_chunk_size` parameter.

{% highlight text %}
root@hinata:~ # hpvmmodify -P batman -x amr_chunk_resize=512
{% endhighlight %}

As in the system-wide parameter the recommendation is to set the chunk size to a multiple of the dynamic memory chunks size.

Finally to display the configuration and the current use of the virtual machines resource entitlements use `hpvmstatus -r`.

{% highlight text %}
root@hinata:~ # hpvmstatus -r
[Virtual Machine Resource Entitlement]
[Virtual CPU entitlement]
 Percent       Cumulative
Virtual Machine Name VM #  #VCPUs Entitlement Maximum   Usage            Usage
==================== ===== ====== =========== ======= ======= ================
rh-www                   1      4       50.0%  100.0%    0.0%                0
sql-dev                  2      4       50.0%  100.0%    0.3%         21611866
rhino                    3      4       50.0%  100.0%    0.0%                0
batman                   4      8       20.0%  100.0%    0.8%          1318996
robin                    5      8       20.0%  100.0%    0.8%            97993
falcon                   6      2       10.0%  100.0%    0.0%                0

[Virtual Machine Memory Entitlement]
 DynMem  Memory   DynMem  DynMem DynMem  Comfort Total    Free   Avail    Mem    AMR     AMR
Virtual Machine Name  VM #   Min   Entitle   Max    Target Current   Min   Memory  Memory  Memory  Press  Chunk   State
==================== ===== ======= ======= ======= ======= ======= ======= ======= ======= ======= ===== ======= ========
rh-www                   1   512MB     0MB     8GB  8192MB  8192MB     0MB     8GB     0MB     0MB   0       0MB DISABLED
sql-dev                  2   512MB     0MB     8GB  8192MB  8192MB     0MB     8GB     0MB     0MB   0       0MB DISABLED
rhino                    3  1024MB  1500MB     6GB  2048MB  6144MB     0MB     6GB     0MB     0MB   0     256MB  ENABLED
batman                   4  1024MB  1500MB     4GB  4090MB  4090MB  1850MB     4GB  2214MB   500MB   0     256MB  ENABLED
robin                    5  1024MB  1500MB     4GB  4090MB  4090MB  1914MB     4GB  2165MB   531MB   0     256MB  ENABLED
falcon                   6   512MB     0MB     6GB  6144MB  6144MB     0MB     6GB     0MB     0MB   0       0MB DISABLED
{% endhighlight %}

I hope this helps to clarify how HPVM manage the memory of the virtual machines and how to customize its configuration. As always any comment would be welcome :-)

Juanma.
