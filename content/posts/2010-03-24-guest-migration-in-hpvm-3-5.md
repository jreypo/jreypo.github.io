---
title: Guest migration in HPVM 3.5
date: 2010-03-24
tags:
- hp-servers
- hp-ux
- sysadmin
showComments: true
---

As I already said many times my current HPVM version is 3.5 so it doesn't support guest online migration. But lacking the online migration feature doesn't mean that we can not perform Integrity VM migration between hosts.

Currently there are two methods to perform migrations:

- HPVM commands.
- MC/ServiceGuard.

In this post I will only cover the HPVM way. I will leave HPVM ServiceGuard clusters for a future post but as many of you already know moving a guest between cluster nodes is like moving any other ServiceGuard package since the guests are managed by SG as packages.

## PREREQUISITES

There is a certain list of prerequisites the guest has to met in order to be successfully migrated between hosts.

- Off-line state:

This is pretty obvious of course, the guest must be off.

- SSH configuration:

In both hosts root must have SSH access through public key authentication to the other.

```text
root@ivmcl01:~ # hpvmmigrate -P hpvm1 -h ivmcl02
hpvmmigrate: SSH execution error. Make sure ssh is setup right on both source and target systems.
```

- Shared devices:

If the guest has a shared device like the CD/DVD of the host, the device has to be deleted from the guest configuration.

```text
root@ivmcl01:~ # hpvmmigrate -P hpvm1 -h ivmcl02
hpvmmigrate: Device /dev/rdsk/c1t4d0 is shared.  Guest with shared storage devices cannot be migrated.
```

- Storage devices:

There are two consideration about storage devices.

The storage devices of the guest must be physical disks. Migration of guests with lvols as storage devices is supported only in [HPVM 4.1](http://h20000.www2.hp.com/bizsupport/TechSupport/CoreRedirect.jsp?redirectReason=DocIndexPDF&prodSeriesId=4146132&targetPage=http%3A%2F%2Fbizsupport1.austin.hp.com%2Fbc%2Fdocs%2Fsupport%2FSupportManual%2Fc02018680%2Fc02018680.pdf "HPVM 4.1 Release Notes") release.

```text
root@ivmcl01:~ # hpvmmigrate -P hpvm1 -h ivmcl02
hpvmmigrate: Target VM Host error - Device does not exist.
hpvmmigrate: See HPVM command log file on target VM Host for more detail.
```

The WWID of the device must be the same in both HPVM hosts.

```text
root@ivmcl01:~ # hpvmmigrate -P hpvm1 -h ivmcl02
hpvmmigrate: Device WWID does not match.
hpvmmigrate: See HPVM command log file on target VM Host for more detail.
```

- Network configuration:

The virtual switch where the guest is connected to must be configured on the same network card in both hosts. For example if vSwitch `vlan2` is using `lan0` in host1 then it must be using `lan0` in host2 or the migration will fail.

```text
root@ivmcl01:~ # hpvmmigrate -P hpvm1 -h ivmcl02
hpvmmigrate: Target VM Host error - vswitch validation failed.
hpvmmigrate: See HPVM command log file on target VM Host for more detail.
```

## PROCEDURE

If all the prerequisites explained before are met by our guest we can proceed with the migration. The command to use is `hpvmmigrate`, the name or the VM number and the hostname of the destination server have to be provided. Some of the resources of the virtual machines like number of CPU, amount of RAM or the machine label can also be modified.

```text
root@ivmcl01:~ # hpvmmigrate -P hpvm1 -h ivmcl02
hpvmmigrate: Guest migrated successfully.
root@ivmcl01:~ #
```

Check the existence of the migrated guest in the destination host.

```text
root@ivmcl02:~ # hpvmstatus
[Virtual Machines]
Virtual Machine Name VM #  OS Type State     #VCPUs #Devs #Nets Memory  Runsysid
==================== ===== ======= ========= ====== ===== ===== ======= ========
oratest01                1 HPUX    On (OS)        4    10     3   16 GB        0
oratest02                2 HPUX    On (OS)        4     8     3   16 GB        0
sapvm01                  3 HPUX    Off            3     8     3    8 GB        0
sapvm02                  4 HPUX    Off            3     7     3    8 GB        0
sles01                   5 LINUX   On (OS)        1     4     3    4 GB        0
rhel01                   6 LINUX   Off            1     4     3    4 GB        0
hp-vxvm                  7 HPUX    On (OS)        2    17     3    6 GB        0
ws2003                   8 WINDOWS Off            4     4     3   12 GB        0
hpvm1                   10 HPUX    Off            1     1     1    3 GB        0
root@ivmcl02:~ #
```

As you can see once all the prerequisites have been met the migration is quite easy.

## CONCLUSION

Even with the disadvantage of lacking online migration the guest migration feature can be of usefulness to balance the load between HPVM hosts.

Juanma.
