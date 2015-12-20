---
layout: post
title: Howto enable HP-UX 11iv3 agile naming mode in VxVM
date: 2010-06-18 11:15:11.000000000 +02:00
type: post
published: true
status: publish
categories:
- HP-UX
- Storage
- Sysadmin
tags:
- 11iv3
- HP-UX
- Storage
- sysadmin
- systems administration
- VxVM
meta:
  _edit_last: '11044808'
  geo_latitude: '40.416691'
  geo_longitude: '-3.7003449999999702'
  geo_accuracy: '17077'
  geo_address: Cortes, Madrid, Madrid, Spain
  geo_public: ''
  reddit: a:2:{s:5:"count";s:1:"0";s:4:"time";s:10:"1331008157";}
  _wpas_skip_twitter: '1'
  _wpas_skip_tumblr: '1'
  _wpas_skip_path: '1'
author: juan_manuel_rey
---

By default Veritas Volume Manager uses HP-UX legacy naming scheme instead of the agile view one, of course for any HP-UX Sysadmin this is completely unacceptable. Below is a small procedure to change it.

Display VxVM disk information  and get the current naming scheme.

{% highlight text %}
root@robin:~# vxdisk list
DEVICE       TYPE            DISK         GROUP        STATUS
c0t0d0s2     auto:LVM        -            -            LVM
c0t1d0       auto:LVM        -            -            LVM
c0t2d0       auto:cdsdisk    labdg01      labdg        online
c0t3d0       auto:cdsdisk    labdg02      labdg        online
c0t4d0       auto:cdsdisk    labdg03      labdg        online
c0t5d0       auto:none       -            -            online invalid
c0t6d0       auto:none       -            -            online invalid
c0t7d0       auto:none       -            -            online invalid
c0t8d0s2     auto:hpdisk     rootdisk01   rootdg       online
c0t9d0s2     auto:hpdisk     rootdisk02   rootdg       online
root@robin:~#
root@robin:~# vxddladm get namingscheme
NAMING_SCHEME       PERSISTENCE         MODE                
===============================================
OS Native           Yes                 Legacy              
root@robin:~#
{% endhighlight %}

As you can see the mode is se tot legacy and the disks are shown with their legacy device names. To change this use again `vxddladm` command.

{% highlight text %}
root@robin:~# vxddladm set namingscheme=osn mode=new
{% endhighlight %}

The parameter used are `namingscheme` and `mode`. The available options for the first are:

-   `ebn` - Enclosure based names.
-   `osn` - Operative system names.

If `ebn` is used neither `legacy` mode nor `new` mode can be set since hardware names provided by the disk array will be used so use `osn` as naming scheme.

The second parameter is mode and of course defines which naming model will be used in the `osn` naming scheme. The following three values can be set:

-   `default`
-   `legacy`
-   `new`

Now check the change by executing `vxdisk` and `vxddladm` commands.

{% highlight text %}
root@robin:~# vxdisk list
DEVICE       TYPE            DISK         GROUP        STATUS
disk4_p2     auto:LVM        -            -            LVM
disk6        auto:LVM        -            -            LVM
disk8        auto:cdsdisk    labdg01      labdg        online
disk10       auto:cdsdisk    labdg02      labdg        online
disk12       auto:cdsdisk    labdg03      labdg        online
disk14       auto:none       -            -            online invalid
disk16       auto:none       -            -            online invalid
disk18       auto:none       -            -            online invalid
disk20_p2    auto:hpdisk     rootdisk01   rootdg       online
disk22_p2    auto:hpdisk     rootdisk02   rootdg       online
root@robin:~#
root@robin:~# vxddladm get namingscheme
NAMING_SCHEME       PERSISTENCE         MODE                
===============================================
OS Native           Yes                 New                 
root@robin:~#
{% endhighlight %}

Of course the naming scheme can be set back to the legacy scheme using the same procedure.

Juanma.
