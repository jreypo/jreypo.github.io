---
layout: post
title: HP ProLiant servers management with hpasmcli
date: 2011-02-16
type: post
published: true
status: publish
categories:
- DevOps
- HP
- Linux
- Sysadmin
meta: juan_manuel_rey
comments: true
---

`hpasmcli`, HP Management Command Line Interface, is a scriptable command line tool to manage and monitor the HP ProLiant servers through the `hpasmd` and `hpasmxld` daemons. It is part of the hp-health package that comes with the [**HP Proliant Support Pack**](http://h18000.www1.hp.com/products/servers/management/psp/index.html), or **PSP**.

{% highlight text %}
[root@rhel4 ~]# rpm -qa | grep hp-health
hp-health-8.1.1-14.rhel4
[root@rhel4 ~]#
[root@rhel4 ~]# rpm -qi hp-health-8.1.1-14.rhel4
Name        : hp-health                    Relocations: (not relocatable)
Version     : 8.1.1                             Vendor: Hewlett-Packard Company
Release     : 14.rhel4                      Build Date: Fri 04 Jul 2008 07:04:51 PM CEST
Install Date: Thu 02 Apr 2009 05:10:48 PM CEST      Build Host: rhel4ebuild.M73C253-lab.net
Group       : System Environment            Source RPM: hp-health-8.1.1-14.rhel4.src.rpm
Size        : 1147219                          License: 2008 Hewlett-Packard Development Company, L.P.
Signature   : (none)
Packager    : Hewlett-Packard Company
URL         : http://www.hp.com/go/proliantlinux
Summary     : hp System Health Application and Command line Utility Package
Description :
This package contains the System Health Monitor for all hp Proliant systems
with ASM, ILO, & ILO2 embedded management asics.  Also contained are the
command line utilities.
[root@rhel4 ~]#
[root@rhel4 ~]# rpm -ql hp-health-8.1.1-14.rhel4
/etc/init.d/hp-health
/opt/hp/hp-health
/opt/hp/hp-health/bin
/opt/hp/hp-health/bin/IrqRouteTbl
/opt/hp/hp-health/bin/hpasmd
/opt/hp/hp-health/bin/hpasmlited
/opt/hp/hp-health/bin/hpasmpld
/opt/hp/hp-health/bin/hpasmxld
/opt/hp/hp-health/hprpm.xpm
/opt/hp/hp-health/sh
/opt/hp/hp-health/sh/hpasmxld_reset.sh
/sbin/hpasmcli
/sbin/hpbootcfg
/sbin/hplog
/sbin/hpuid
/usr/lib/libhpasmintrfc.so
/usr/lib/libhpasmintrfc.so.2
/usr/lib/libhpasmintrfc.so.2.0
/usr/lib/libhpev.so
/usr/lib/libhpev.so.1
/usr/lib/libhpev.so.1.0
/usr/lib64/libhpasmintrfc64.so
/usr/lib64/libhpasmintrfc64.so.2
/usr/lib64/libhpasmintrfc64.so.2.0
/usr/share/man/man4/hp-health.4.gz
/usr/share/man/man4/hpasmcli.4.gz
/usr/share/man/man7/hp_mgmt_install.7.gz
/usr/share/man/man8/hpbootcfg.8.gz
/usr/share/man/man8/hplog.8.gz
/usr/share/man/man8/hpuid.8.gz
[root@rhel4 ~]#
{% endhighlight %}

This handy tool can be used to view and modify several BIOS settings of the server and to monitor the status of the different hardware components like fans, memory modules, temperature, power supplies, etc.

It can be used in two ways:

-   Interactive shell
-   Within a script

The interactive shell supports TAB command completion and command recovery through a history buffer.

{% highlight text %}
[root@rhel4 ~]# hpasmcli
HP management CLI for Linux (v1.0)
Copyright 2004 Hewlett-Packard Development Group, L.P.

--------------------------------------------------------------------------
NOTE: Some hpasmcli commands may not be supported on all Proliant servers.
      Type 'help' to get a list of all top level commands.
--------------------------------------------------------------------------
hpasmcli> help
CLEAR  DISABLE  ENABLE  EXIT  HELP  NOTE  QUIT  REPAIR  SET  SHOW
hpasmcli>
{% endhighlight %}

As it can be seen in the above example several main tasks can be done, to get the usage of every command simply use HELP followed by the command.

{% highlight text %}
hpasmcli> help show
USAGE: SHOW [ ASR | BOOT | DIMM | F1 | FANS | HT | IML | IPL | NAME | PORTMAP | POWERSUPPLY | PXE | SERIAL | SERVER | TEMP | UID | WOL ]
hpasmcli>
hpasmcli> HELP SHOW BOOT
USAGE: SHOW BOOT: Shows boot devices.
hpasmcli>
{% endhighlight %}

In my experience `SHOW` is the most used command above the others. Following are examples for some of the tasks.

#### Display general information of the server

{% highlight text %}
hpasmcli> SHOW SERVER
System        : ProLiant DL380 G5
Serial No.    : XXXXXXXXX     
ROM version   : P56 11/01/2008
iLo present   : Yes
Embedded NICs : 2
        NIC1 MAC: 00:1c:c4:62:42:a0
        NIC2 MAC: 00:1c:c4:62:42:9e

Processor: 0
        Name         : Intel Xeon
        Stepping     : 11
        Speed        : 2666 MHz
        Bus          : 1333 MHz
        Core         : 4
        Thread       : 4
        Socket       : 1
        Level2 Cache : 8192 KBytes
        Status       : Ok

Processor: 1
        Name         : Intel Xeon
        Stepping     : 11
        Speed        : 2666 MHz
        Bus          : 1333 MHz
        Core         : 4
        Thread       : 4
        Socket       : 2
        Level2 Cache : 8192 KBytes
        Status       : Ok

Processor total  : 2

Memory installed : 16384 MBytes
ECC supported    : Yes
hpasmcli>
{% endhighlight %}

#### Show current temperatures

{% highlight text %}
hpasmcli> SHOW TEMP
Sensor   Location              Temp       Threshold
------   --------              ----       ---------
#1        I/O_ZONE             49C/120F   70C/158F
#2        AMBIENT              23C/73F    39C/102F
#3        CPU#1                30C/86F    127C/260F
#4        CPU#1                30C/86F    127C/260F
#5        POWER_SUPPLY_BAY     52C/125F   77C/170F
#6        CPU#2                30C/86F    127C/260F
#7        CPU#2                30C/86F    127C/260F

hpasmcli>
{% endhighlight %}

#### Get the status of the server fans

{% highlight text %}
hpasmcli> SHOW FAN
Fan  Location        Present Speed  of max  Redundant  Partner  Hot-pluggable
---  --------        ------- -----  ------  ---------  -------  -------------
#1   I/O_ZONE        Yes     NORMAL 45%     Yes        0        Yes          
#2   I/O_ZONE        Yes     NORMAL 45%     Yes        0        Yes          
#3   PROCESSOR_ZONE  Yes     NORMAL 41%     Yes        0        Yes          
#4   PROCESSOR_ZONE  Yes     NORMAL 36%     Yes        0        Yes          
#5   PROCESSOR_ZONE  Yes     NORMAL 36%     Yes        0        Yes          
#6   PROCESSOR_ZONE  Yes     NORMAL 36%     Yes        0        Yes          

hpasmcli>
{% endhighlight %}

#### Show device boot order configuration

{% highlight text %}
hpasmcli> SHOW BOOT
First boot device is: CDROM.
One time boot device is: Not set.
hpasmcli>
{% endhighlight %}

#### Set USB key as first boot device

{% highlight text %}
hpasmcli> SET BOOT FIRST USBKEY
{% endhighlight %}

#### Show memory modules status

{% highlight text %}
hpasmcli> SHOW DIMM
DIMM Configuration
------------------
Cartridge #:   0
Module #:      1
Present:       Yes
Form Factor:   fh
Memory Type:   14h
Size:          4096 MB
Speed:         667 MHz
Status:        Ok

Cartridge #:   0
Module #:      2
Present:       Yes
Form Factor:   fh
Memory Type:   14h
Size:          4096 MB
Speed:         667 MHz
Status:        Ok

Cartridge #:   0
Module #:      3
Present:       Yes
Form Factor:   fh
Memory Type:   14h
Size:          4096 MB
Speed:         667 MHz
Status:        Ok
...
{% endhighlight %}

In the scripting mode `hpasmcli` can be used directly from the shell prompt with the `-s` option and the command between quotation marks, this of course allow you to process the output of the commands  like in the below example.

{% highlight text %}
[root@rhel4 ~]# hpasmcli -s "show dimm" | egrep "Module|Status"
Module #:      1
Status:        Ok
Module #:      2
Status:        Ok
Module #:      3
Status:        Ok
Module #:      4
Status:        Ok
Module #:      5
Status:        Ok
Module #:      6
Status:        Ok
Module #:      7
Status:        Ok
Module #:      8
Status:        Ok
[root@rhel4 ~]#
{% endhighlight %}

To execute more than one command sequentially separate them with a semicolon.

{% highlight text %}
[root@rhel4 ~]# hpasmcli -s "show fan; show temp"

Fan  Location        Present Speed  of max  Redundant  Partner  Hot-pluggable
---  --------        ------- -----  ------  ---------  -------  -------------
#1   I/O_ZONE        Yes     NORMAL 45%     Yes        0        Yes          
#2   I/O_ZONE        Yes     NORMAL 45%     Yes        0        Yes          
#3   PROCESSOR_ZONE  Yes     NORMAL 41%     Yes        0        Yes          
#4   PROCESSOR_ZONE  Yes     NORMAL 36%     Yes        0        Yes          
#5   PROCESSOR_ZONE  Yes     NORMAL 36%     Yes        0        Yes          
#6   PROCESSOR_ZONE  Yes     NORMAL 36%     Yes        0        Yes          

Sensor   Location              Temp       Threshold
------   --------              ----       ---------
#1        I/O_ZONE             47C/116F   70C/158F
#2        AMBIENT              21C/69F    39C/102F
#3        CPU#1                30C/86F    127C/260F
#4        CPU#1                30C/86F    127C/260F
#5        POWER_SUPPLY_BAY     50C/122F   77C/170F
#6        CPU#2                30C/86F    127C/260F
#7        CPU#2                30C/86F    127C/260F

[root@rhel4 ~]#
{% endhighlight %}

If you want to play more with `hpasmcli` go to its man page and to the ProLiant Support Pack [documentation](http://bizsupport2.austin.hp.com/bc/docs/support/SupportManual/c02532067/c02532067.pdf).

Juanma.
