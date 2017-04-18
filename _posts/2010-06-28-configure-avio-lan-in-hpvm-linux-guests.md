---
layout: post
title: Configure AVIO Lan in HPVM Linux guests
date: 2010-06-28
type: post
published: true
status: publish
categories:
- HP-UX
- Itanium
- Linux
- Sysadmin
- Virtualization
tags:
- AVIO Lan
- HPVM
- HPVM Linux guests
- Integrity VMs
- Linux
- networking
- sysadmin
author: juan_manuel_rey
comments: true
---

The  AVIO Lan drivers for Linux HPVM guests are supported since HPVM4.0 but as you will see enabling it is a little more complicated than in HP-UX guests.

The first prerequisite is to have installed the HPVM management software, once you have this package installed look for a RPM package called `hpvm_lgssn` in `/opt/hpvm/guest-images/linux/DRIVERS`.

```
root@hpvm-host:/opt/hpvm/guest-images/linux/DRIVERS # ll
total 584
 0 drwxr-xr-x 2 bin bin     96 Apr 13 18:47 ./
 0 drwxr-xr-x 5 bin bin     96 Apr 13 18:48 ../
 8 -r--r--r-- 1 bin bin   7020 Mar 27  2009 README
576 -rw-r--r-- 1 bin bin 587294 Mar 27  2009 hpvm_lgssn-4.1.0-3.ia64.rpm
root@hpvm-host:/opt/hpvm/guest-images/linux/DRIVERS #
```

Copy the package to the virtual machine with your favorite method and install it.

```
[sles10]:/var/tmp # rpm -ivh hpvm_lgssn-4.1.0-3.ia64.rpm
Preparing...                ########################################### [100%]
Installing...               ########################################### [100%]

[sles10]:/var/tmp #
```

Check the installation of the package.

```
[sles10]:~ # rpm -qa | grep hpvm
hpvm-4.1.0-1
hpvmprovider-4.1.0-1
hpvm_lgssn-4.1.0-3
[sles10]:~ #
[sles10]:~ # rpm -ql hpvm_lgssn
/opt/hpvm_drivers
/opt/hpvm_drivers/lgssn
/opt/hpvm_drivers/lgssn/LICENSE
/opt/hpvm_drivers/lgssn/Makefile
/opt/hpvm_drivers/lgssn/README
/opt/hpvm_drivers/lgssn/hpvm_guest.h
/opt/hpvm_drivers/lgssn/lgssn.h
/opt/hpvm_drivers/lgssn/lgssn_ethtool.c
/opt/hpvm_drivers/lgssn/lgssn_main.c
/opt/hpvm_drivers/lgssn/lgssn_recv.c
/opt/hpvm_drivers/lgssn/lgssn_recv.h
/opt/hpvm_drivers/lgssn/lgssn_send.c
/opt/hpvm_drivers/lgssn/lgssn_send.h
/opt/hpvm_drivers/lgssn/lgssn_trace.h
/opt/hpvm_drivers/lgssn/rh4
/opt/hpvm_drivers/lgssn/rh4/u5
/opt/hpvm_drivers/lgssn/rh4/u5/lgssn.ko
/opt/hpvm_drivers/lgssn/rh4/u6
/opt/hpvm_drivers/lgssn/rh4/u6/lgssn.ko
/opt/hpvm_drivers/lgssn/sles10
/opt/hpvm_drivers/lgssn/sles10/SP1
/opt/hpvm_drivers/lgssn/sles10/SP1/lgssn.ko
/opt/hpvm_drivers/lgssn/sles10/SP2
/opt/hpvm_drivers/lgssn/sles10/SP2/lgssn.ko
[sles10]:~ #
```

There are two ways to install the driver, compile it or use one of the pre-compiled modules. These pre-compiled modules are for the following distributions and kernels:

-   Red Hat 4 release 5 (2.6.9-55.EL)
-   Red Hat 4 release 6 (2.6.9-67.EL)
-   SLES10 SP1 (2.6.16.46-0.12)
-   SLES10 SP2 (2.6.16.60-0.21)

For other kernels you must compile the driver. In the Linux box of the example I had a supported kernels and distro (SLES10 SP2) but instead of using the pre-compiled one I decided to go through the whole process.

Go the path `/opt/hpvm_drivers/lgssn`, there you will find the sources of the driver. To compile and install execute a simple `make install`.

```
[sles10]:/opt/hpvm_drivers/lgssn # make install
make -C /lib/modules/2.6.16.60-0.21-default/build SUBDIRS=/opt/hpvm_drivers/lgssn modules
make[1]: Entering directory `/usr/src/linux-2.6.16.60-0.21-obj/ia64/default'
make -C ../../../linux-2.6.16.60-0.21 O=../linux-2.6.16.60-0.21-obj/ia64/default modules
 CC [M]  /opt/hpvm_drivers/lgssn/lgssn_main.o
 CC [M]  /opt/hpvm_drivers/lgssn/lgssn_send.o
 CC [M]  /opt/hpvm_drivers/lgssn/lgssn_recv.o
 CC [M]  /opt/hpvm_drivers/lgssn/lgssn_ethtool.o
 LD [M]  /opt/hpvm_drivers/lgssn/lgssn.o
 Building modules, stage 2.
 MODPOST
 CC      /opt/hpvm_drivers/lgssn/lgssn.mod.o
 LD [M]  /opt/hpvm_drivers/lgssn/lgssn.ko
make[1]: Leaving directory `/usr/src/linux-2.6.16.60-0.21-obj/ia64/default'
find /lib/modules/2.6.16.60-0.21-default -name lgssn.ko -exec rm -f {} \; || true
find /lib/modules/2.6.16.60-0.21-default -name lgssn.ko.gz -exec rm -f {} \; || true
install -D -m 644 lgssn.ko /lib/modules/2.6.16.60-0.21-default/kernel/drivers/net/lgssn/lgssn.ko
/sbin/depmod -a || true
[sles10]:/opt/hpvm_drivers/lgssn #
```

This will copy the driver to `/lib/module/<KERNEL_VERSION>/kernel/drivers/net/lgssn/`.

To ensure that the new driver will loaded during the startup of the operative system first add the following line to `/etc/modprobe.conf`, one line for each interface configured for AVIO Lan.

```
alias eth1 lgssn
```

The HPVM 4.2 manual said you have to issue the command `depmod -a` in order to inform the kernel about the change but if you look the above log will see that the last command executed by the make install is a `depmod -a`. Look into the `modules.dep` file to check that the corresponding line for the `lgssn` driver has been added.

```
[sles10]:~ # grep lgssn /lib/modules/2.6.16.60-0.21-default/modules.dep
/lib/modules/2.6.16.60-0.21-default/kernel/drivers/net/lgssn/lgssn.ko:
[sles10]:~ #
```

At this point and if you have previously reconfigured the virtual machine, load the module and restart the network services.

```
[sles10]:/opt/hpvm_drivers/lgssn # insmod /lib/modules/2.6.16.60-0.21-default/kernel/drivers/net/lgssn/lgssn.ko
[sles10]:/opt/hpvm_drivers/lgssn # lsmod |grep lgssn
lgssn                 576136  0
[sles10]:/opt/hpvm_drivers/lgssn #
[sles10]:/opt/hpvm_drivers/lgssn # service network restart
Shutting down network interfaces:
    eth0      device: Intel Corporation 82540EM Gigabit Ethernet Controller
    eth0      configuration: eth-id-2a:87:14:5c:f9:ed
    eth0                                                              done
    eth1      device: Hewlett-Packard Company Unknown device 1338
    eth1      configuration: eth-id-66:f3:f8:4e:37:d5
    eth1                                                              done
    eth2      device: Intel Corporation 82540EM Gigabit Ethernet Controller
    eth2      configuration: eth-id-0a:dc:fd:cb:2c:62
    eth2                                                              done
Shutting down service network  .  .  .  .  .  .  .  .  .  .  .  .  .  done
Hint: you may set mandatory devices in /etc/sysconfig/network/config
Setting up network interfaces:
    lo        
    lo       
              IP address: 127.0.0.1/8   
              IP address: 127.0.0.2/8   
Checking for network time protocol daemon (NTPD):                     running
    lo                                                                done
    eth0      device: Intel Corporation 82540EM Gigabit Ethernet Controller
    eth0      configuration: eth-id-2a:87:14:5c:f9:ed
Warning: Could not set up default route via interface
 Command ip route replace to default via 10.31.12.1 returned:
 . RTNETLINK answers: Network is unreachable
 Configuration line: default 10.31.12.1 - -
 This needs NOT to be AN ERROR if you set up multiple interfaces.
 See man 5 routes how to avoid this warning.

Checking for network time protocol daemon (NTPD):                     running
    eth0                                                              done
    eth1      device: Hewlett-Packard Company Unknown device 1338
    eth1      configuration: eth-id-66:f3:f8:4e:37:d5
    eth1      IP address: 10.31.4.16/24   
Warning: Could not set up default route via interface
 Command ip route replace to default via 10.31.12.1 returned:
 . RTNETLINK answers: Network is unreachable
 Configuration line: default 10.31.12.1 - -
 This needs NOT to be AN ERROR if you set up multiple interfaces.
 See man 5 routes how to avoid this warning.

Checking for network time protocol daemon (NTPD):                     running
    eth1                                                              done
    eth2      device: Intel Corporation 82540EM Gigabit Ethernet Controller
    eth2      configuration: eth-id-0a:dc:fd:cb:2c:62
    eth2      IP address: 10.31.12.11/24   
Checking for network time protocol daemon (NTPD):                     running
    eth2                                                              done
Setting up service network  .  .  .  .  .  .  .  .  .  .  .  .  .  .  done
[sles10]:/opt/hpvm_drivers/lgssn #
```

If you have not configured the networking interface of the virtual machine shutdown the virtual machine and from the host modify each virtual NIC of the guest. Take into account that AVIO LAN drivers are not supported with `localnet` virtual switches.

```
root@hpvm-host:~ # hpvmmodify -P sles10 -m network:avio_lan:0,2:vswitch:vlan2:portid:4
root@hpvm-host:~ # hpvmstatus -P sles10 -d
[Virtual Machine Devices]
...
[Network Interface Details]
network:lan:0,0,0x2A87145CF9ED:vswitch:localnet:portid:4
network:avio_lan:0,1,0x66F3F84E37D5:vswitch:vlan1:portid:4
network:avio_lan:0,2,0x0ADCFDCB2C62:vswitch:vlan2:portid:4
...
root@hpvm-host:~ #
```

Finally start the virtual machine and check that everything went well and the drivers have been loaded.

Juanma
