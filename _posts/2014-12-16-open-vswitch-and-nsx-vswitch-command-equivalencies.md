---
title: Open vSwitch and NSX vSwitch command equivalencies
date: 2014-12-16
type: post
classes: wide
published: true
status: publish
categories:
- Networking
- Sysadmin
- VMware
tags:
- network virtualization
- networking
- NSX
- NSX vSwitch
- NSX-MH
- NVS
- Open vSwitch
- OVS
- VMware
author: juan_manuel_rey
comments: true
---

A question I've heard a few times, what are the command equivalencies between a standard **Open vSwitch**, running inside a Linux box, and the **NSX vSwitch** running inside ESXi? I have written this post to clarify this a bit.

There are four commands in NSX CLI that have equivalencies in the OVS
world:

  **NVS Command** | **OVS Command**
  :---: | :---:
  nsx-dbctl | ovs-vsctl
  nsx-dpctl | ovs-dpctl
  nsx-appctl | ovs-appctl
  nsx-flowctl | ovs-flowctl

## nsx-dbctl

`nsx-dbctl` command, like its OVS equivalent `ovs-vsctl`, sub-commands are the same, and for example `nsx-dbctl show` will produce a similar output to `ovs-vsctl show`.

```text
~ # nsx-dbctl show
ec451c1a-0258-423a-b406-dec83af4b241
    Manager "ssl:192.168.110.201:6632"
        is_connected: true
    Bridge "br-vmnic1"
        fail_mode: standalone
        Port "vmk3"
            Interface "vmk3"
        Port "vmnic1"
            Interface "vmnic1"
    Bridge br-int
        Controller "ssl:192.168.110.201:6633"
            is_connected: true
        Controller "unix:ovs-l3d.mgmt"
            is_connected: true
        fail_mode: secure
        Port "vNic.3000004"
            Interface "vNic.3000004"
        Port "vNic.3000006"
            Interface "vNic.3000006"
        Port "vNic.3000005"
            Interface "vNic.3000005"
    ovs_version: "2.0.2.31704"
~ #
```

## nsx-dpctl

`nsx-dpctl` command maps to `ovs-dpctl` and much like it allow you to manage Open vSwitch datapaths.

```text
~ # nsx-dpctl show
system@nsx-vswitch:
        lookups: hit:1770781 missed:192476 lost:0
        flows: 14
        port 50331650: vmnic1
        port 50331651: vmk3
        port 50331652: vNic.3000004
        port 50331653: vNic.3000005
        port 50331654: vNic.3000006
~ #
```

## nsx-appctl

`nsx-appctl` will allow the administrator to manage and configure OVS daemons. It maps to `ovs-appctl` command.

```text
~ # nsx-appctl dpif/show
system@nsx-vswitch: hit:2230477 missed:148652
        flows: cur: 17, avg: 17, max: 33, life span: 1918447ms
        hourly avg: add rate: 66.907/min, del rate: 66.880/min
        daily avg: add rate: 43.476/min, del rate: 43.461/min
        overall avg: add rate: 60.918/min, del rate: 60.909/min
        br-int: hit:142949 missed:8461
                vNic.3000004 1/50331652: (system)
                vNic.3000005 2/50331653: (system)
                vNic.3000006 3/50331654: (system)
        br-vmnic1: hit:2087528 missed:140191
                vmk3 2/50331651: (system)
                vmnic1 1/50331650: (system)
~ #
```

## nsx-flowctl

`nsx-flowctl` is the equivalent of `ovs-flowctl` and will allow you to manage NSX vSwich flow tables, ports, etc.

```text
~ # nsx-flowctl show br-bond0
OFPT_FEATURES_REPLY (xid=0x3): dpid:0000725d4492c540
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: OUTPUT SET_VLAN_VID SET_VLAN_PCP STRIP_VLAN SET_DL_SRC SET_DL_DST SET_NW_SRC SET_NW_DST SET_NW_TOS SET_TP_SRC SET_TP_DST ENQUEUE
 1(vmnic4): addr:00:50:56:01:08:c6
     config:     0
     state:      0
     current:    1GB-FD
     advertised: 10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-HD 1GB-FD
     supported:  10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-HD 1GB-FD
     speed: 1000 Mbps now, 1000 Mbps max
 2(vmnic5): addr:00:50:56:01:08:c8
     config:     0
     state:      0
     current:    1GB-FD
     advertised: 10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-HD 1GB-FD
     supported:  10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-HD 1GB-FD
     speed: 1000 Mbps now, 1000 Mbps max
 3(vmk3): addr:00:50:56:66:57:18
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x6): frags=normal miss_send_len=0
~ #
```

Courteous comments are welcome.

Juanma.
