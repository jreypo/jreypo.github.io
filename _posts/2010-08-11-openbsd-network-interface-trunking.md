---
title: OpenBSD network interface trunking
date: 2010-08-11
type: post
classes: wide
published: true
status: publish
categories:
- BSD
- Networking
- Sysadmin
tags:
- LACP
- networking
- OpenBSD
- sysadmin
- systems administration
- trunk
author: juan_manuel_rey
comments: true
---

Network interface trunking is the OpenBSD equivalent of HP-UX Auto-Port Aggregation feature. It allows to combine two or more physical interfaces into a virtual one that will send the outgoing traffic through the physical ports with an algorithm that depends on the trunking protocol configured.

The trunk driver has been available since OpenBSD 3.8, then it only supported the Round-robin protocol, and in the current version, OpenBSD 4.7, it supports the following protocols:

- **Broadcast:** Sends frames to all ports of the trunk and equally receives frames from any port.
- **Round-robin:** Distributes outgoing traffic through all active ports and accepts incoming traffic from any active port.
- **Failover:** Sends and receives traffic only through the master port. If the master port becomes unavailable, the next active port is used. The first interface added is the master port; any interfaces added after that are used as failover devices.
- **Loadbalance:** The Loadbalance protocol balances the outgoing traffic across the active ports based on hashed protocol header information and accepts incoming traffic from any active port. The hash includes the Ethernet source and destination address, and, if available, the VLAN tag, and the IP source and destination address.
- **LACP:** Used to provide redundancy and increase link speed, it uses the IEEE 802.3ad Link Aggregation Control Protocol (LACP) and the Marketer Protocol. It requires a LACP capable switch.
- **None:** This protocol disables any traffic without disabling the trunk interface itself.

Lets create a failover trunk interface as an example. First we are going to activate the physical interfaces and add them to the `trunk0` interface.

```
[obsd47]/# ifconfig em1 up
[obsd47]/# ifconfig em2 up
[obsd47]/# ifconfig trunk0 trunkport em1
[obsd47]/# ifconfig trunk0 trunkport em2
[obsd47]/# ifconfig trunk0
trunk0: flags=8802<BROADCAST,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:0c:29:24:b7:c6
        priority: 0
        trunk: trunkproto roundrobin
                trunkport em2 active
                trunkport em1 master,active
        groups: trunk
        media: Ethernet autoselect
        status: active
[obsd47]/#
```

Configure the trunking protocol and the IP address of the interface.

```
[obsd47]/# ifconfig trunk0 trunkproto failover 192.168.126.5 netmask 255.255.255.0 up
[obsd47]/# ifconfig
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 33200
        priority: 0
        groups: lo
        inet 127.0.0.1 netmask 0xff000000
        inet6 ::1 prefixlen 128
        inet6 fe80::1%lo0 prefixlen 64 scopeid 0x5
em0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:0c:29:24:b7:bc
        priority: 0
        media: Ethernet autoselect (1000baseT full-duplex,master)
        status: active
        inet 192.168.126.4 netmask 0xffffff00 broadcast 192.168.126.255
        inet6 fe80::20c:29ff:fe24:b7bc%em0 prefixlen 64 scopeid 0x1
em1: flags=8b43<UP,BROADCAST,RUNNING,PROMISC,ALLMULTI,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:0c:29:24:b7:c6
        priority: 0
        trunk: trunkdev trunk0
        media: Ethernet autoselect (1000baseT full-duplex,master)
        status: active
        inet6 fe80::20c:29ff:fe24:b7c6%em1 prefixlen 64 scopeid 0x2
em2: flags=8b43<UP,BROADCAST,RUNNING,PROMISC,ALLMULTI,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:0c:29:24:b7:c6
        priority: 0
        trunk: trunkdev trunk0
        media: Ethernet autoselect (1000baseT full-duplex,master)
        status: active
        inet6 fe80::20c:29ff:fe24:b7d0%em2 prefixlen 64 scopeid 0x3
enc0: flags=0<> mtu 1536
        priority: 0
trunk0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:0c:29:24:b7:c6
        priority: 0
        trunk: trunkproto failover
                trunkport em2
                trunkport em1 master,active
        groups: trunk
        media: Ethernet autoselect
        status: active
        inet 192.168.126.5 netmask 0xffffff00 broadcast 192.168.126.255
        inet6 fe80::20c:29ff:fe24:b7c6%trunk0 prefixlen 64 scopeid 0x6
pflog0: flags=141<UP,RUNNING,PROMISC> mtu 33200
        priority: 0
        groups: pflog
[obsd47]/#
```

At this point we have a configured trunk interface in failover, but of course we want to make these changes persistent through a reboot of the server. We need to create a configuration file for each of the physical interfaces and another one for the trunk interface.

```
[obsd47]/# echo "up" >hostname.em1
[obsd47]/# echo "up" >hostname.em2
[obsd47]/# echo "trunkproto failover trunkport em1 trunkport em2 192.168.126.5 netmask 255.255.255.0" > hostname.trunk0
[obsd47]/#
[obsd47]/# cat hostname.trunk0
trunkproto failover trunkport em1 trunkport em2 192.168.126.5 netmask 255.255.255.0
[obsd47]/#
```

Now reboot and check that everything went well and the `trunk0` interface is up and running. Of course the same procedure can be used to create a trunk interface for any of the supported protocols.

Juanma.
