---
title: Moving vNICs between vSwitches
date: 2010-03-03
categories:
- HP-UX
- Itanium
- Networking
- Sysadmin
- Virtualization
tags:
- HP-UX
- HPVM
- Integrity VMs
- networking
- sysadmin
- systems administration
showComments: true
---

Following with my re-learning **HPVM** process today I've been playing around with my virtual switches and a question had arise.

How can I move a virtual Nic from one vSwitch to another?

I discovered is not a difficult task, just one important question to take into account, the virtual machine must be powered off. This kind of changes can't be done if the IVM is online, at least with HPVM 3.5. I never used 4.0 or 4.1 releases of HPVM and I didn't find anything in the documentation that suggest a different behavior.

To perform the operation we're going to use, as usual ;-), `hpvmmodify`. It comes with the `-m` switch to modify the I/O resources of an already existing virtual machine, but you have to specify the hardware address of the device. To identify the address of the network card  launch `hpvmstatus` with  `-d`, this options shows the output with the format used on the command line.

```text
[root@hpvmhost] ~ # hpvmstatus -P ivm1 -d
[Virtual Machine Devices]
...
[Network Interface Details]
network:lan:0,0,0x56E9E3096A22:vswitch:vlan02
network:lan:0,1,0xAED6F7FA4E3E:vswitch:localnet
...
[root@hpvmhost] ~ #
```

As it can be seen in the Networking Interface Details the third field shows, separated by commas,  the lan bus, the device number and the MAC address of the `vNic`. We only need the first two values, that is the lan bus and device number, `0,0` in our the example.

Now we can proceed.

```text
[root@hpvmhost] ~ # hpvmmodify -P ivm2 -m network:lan:0,0:vswitch:vlan03   
[root@hpvmhost] ~ #
[root@hpvmhost] ~ # hpvmstatus -P ivm1
[Virtual Machine Details]
Virtual Machine Name VM #  OS Type State
==================== ===== ======= ========
ivm1                     9 HPUX    On (OS)   
...
[Network Interface Details]
Interface Adaptor    Name/Num   PortNum Bus Dev Ftn Mac Address
========= ========== ========== ======= === === === =================
vswitch   lan        vlan03     9         0   0   0 56-e9-e3-09-6a-22
vswitch   lan        localnet   9         0   1   0 ae-d6-f7-fa-4e-3e
...
[root@hpvmhost] ~ #
```

And we are done.

I will write a few additional posts covering  more HPVM tips, small ones and big ones, at the same time I'm practicing them on my lab server.

Juanma.
