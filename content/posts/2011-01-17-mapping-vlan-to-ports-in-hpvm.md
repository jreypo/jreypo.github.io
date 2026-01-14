---
title: Mapping VLAN to ports in HPVM
date: 2011-01-17
tags:
- hp-servers
- hp-ux
- networking
- sysadmin
showComments: true
---

Long time since my last post about [HP Integrity Virtual Machines](http://h20338.www2.hp.com/enterprise/us/en/os/hpux11i-partitioning-integrity-vm.html), well you know I've been very occupied with vSphere and Linux but that doesn't mean that I completely eliminate HP-UX from my life, on the contrary... **HP-UX ROCKS! :-D**

This is just a quick post on how to map a specific port of virtual switch to a specific VLAN. First retrieve the configuration of the vswitch.

```text
[root@hpvmhost] ~ # hpvmnet -S devlan12
Name     Number State   Mode      NamePPA  MAC Address    IP Address
======== ====== ======= ========= ======== ============== ===============
devlan12      3 Up      Shared    lan4     0x000cfc0046b9 10.1.1.99   

[Port Configuration Details]
Port    Port         Port     Untagged Number of    Active VM
Number  State        Adaptor  VLANID   Reserved VMs
======= ============ ======== ======== ============ ============
1       Active       lan      none     1            oradev01
2       Active       lan      none     1            oradev02
3       Active       lan      none     1            oradev03
4       Active       lan      none     1            oradev04
5       Active       lan      none     1            nfstest01
6       Active       lan      none     1            linuxvm1
7       Active       lan      none     1            linuxvm2

[root@hpvmhost] ~ #
```

We are going to map the port 5 to the VLAN 120 in order to isolate the traffic of that NFS server from the other virtual machines that aren't on the same VLAN. Again the command to use is `hpmvnet`.

```text
[root@hpvmhost] ~ # hpvmnet -S devlan12 -u portid:5:vlanid:120
```

If you display again the HPVM network configuration for the `devlan12` vswitch the change will appear under the `Untagged VLANID` column.

```text
[root@hpvmhost] ~ # hpvmnet -S devlan12
Name     Number State   Mode      NamePPA  MAC Address    IP Address
======== ====== ======= ========= ======== ============== ===============
devlan12      3 Up      Shared    lan4     0x000cfc0046b9 10.1.1.99   

[Port Configuration Details]
Port    Port         Port     Untagged Number of    Active VM
Number  State        Adaptor  VLANID   Reserved VMs
======= ============ ======== ======== ============ ============
1       Active       lan      none     1            oradev01
2       Active       lan      none     1            oradev02
3       Active       lan      none     1            oradev03
4       Active       lan      none     1            oradev04
5       Active       lan      120      1            nfstest01
6       Active       lan      none     1            linuxvm1
7       Active       lan      none     1            linuxvm2

[root@hpvmhost] ~ #
```

Juanma.
