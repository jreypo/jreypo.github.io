---
title: Configuring APA
date: 2010-01-11
type: post
classes: wide
published: true
status: publish
categories:
- HP-UX
- Networking
- Sysadmin
tags:
- 11iv3
- APA
- HP-UX
- nwmgr
- SMH
- sysadmin
- systems administration
author: juan_manuel_rey
comments: true
---

Long time since my last post, Christmas you know ;-) BTW Merry Christmas and Happy New Year to all of you.

During this weeks I've been configuring APA failover groups in some of  my blades and I thought It would be interesting to post how I did it. There are two ways to configure APA in HP-UX 11.31, through the SMH and with the `nwmgr` command. The screenshots are in Spanish since this is how the SMH of my servers are configured.

### SMH

Log into the server SMH and go to **Tools** -> **Network Interface Configuration** -> **Auto Port Aggregation**.

[![SMH Network Tools](/images/smh_network_tools_apa.jpg "SMH APA")]({{site.url}}/images/smh_network_tools_apa.jpg)

Now click on **Create Failover Group** and select the interfaces to add, you can also configure the advanced parameters for the group, when you're done click OK to finish the operation.

[![APA Failover configuration](/images/apa_failover_advanced.jpg "APA Failover configuration")]({{site.url}}/images/apa_failover_advanced.jpg)

Now  in the main page of APA the new group will be shown instead of the LAN interfaces the are part of the group:

[![APA Main Page](/images/apa_finished.jpg "APA Main page")]({{site.url}}/images/apa_finished.jpg)

And this is it, the new APA Failover Group is configured and completely transparent to the users. From this point you can show the information about the newly created APA interface.

[![APA Interface Configuration](/images/apa_showing_interface1.jpg "APA Interface Configuration")]({{site.url}}/images/apa_showing_interface1.jpg)

[![APA Configuration](/images/apa_showing_config1.jpg "APA Configuration")]({{site.url}}/images/apa_showing_config1.jpg)

### nwmgr command

The `nwmgr` command has replaced the old `lanscan`, `lanadmin` and `linkloop` commands in 11.31. I have to say that I prefer this way instead of the SMH one to do any APA or LAN related tasks. SMH require `TCP/2301` and `TCP/2381` ports to be open and accessible and this is not always possible mainly because security policies but to use `nwmgr` you only need access to the server through SSH.

To begin show the current configuration:

```
[root@ignite] ~ # nwmgr -g

Name/          Interface Station          Sub-   Interface      Related
ClassInstance  State     Address        system   Type           Interface
============== ========= ============== ======== ============== =========
lan2           DOWN      0x001E0B45E09A igelan   1000Base-SX
lan0           UP        0x001E0B45E09C igelan   1000Base-SX    lan900
lan1           UP        0x001E0B45E09D igelan   1000Base-SX    lan900
lan3           DOWN      0x001E0B45E09B igelan   1000Base-SX
lan900         UP        0x001E0B45E09C hp_apa   hp_apa
lan901         DOWN      0x000000000000 hp_apa   hp_apa
lan902         DOWN      0x000000000000 hp_apa   hp_apa
lan903         DOWN      0x000000000000 hp_apa   hp_apa
lan904         DOWN      0x000000000000 hp_apa   hp_apa
[root@ignite] ~ # nwmgr -g -S apa
Class    Mode        Load      Speed-               Membership
Instance             Balancing Duplex
======== =========== ========= ==================== ===========================
lan900   Not_Enabled LB_MAC    0 Mbps                -
lan901   Not_Enabled LB_MAC    0 Mbps                -
lan902   Not_Enabled LB_MAC    0 Mbps                -
lan903   Not_Enabled LB_MAC    0 Mbps                -
lan904   Not_Enabled LB_MAC    0 Mbps                -
[root@ignite] ~ #
```

Now proceed with the creation of the new failover group:

```
[root@ignite] ~ # nwmgr -a -S apa -c lan900 -A links=0,1 -A mode=LAN_MONITOR
Addition of links 0, 1 to lan900 succeeded.
[root@ignite] ~ # nwmgr -s -S apa -A all --saved --from cu
[root@ignite] ~ #
```

Specific advanced parameters can also be set, in the example I used the parameters `rapid_arp_count` and `poll_interval`:

```
[root@ignite] / # nwmgr -a -S apa -c lan901 -A links=2,1,3 -A mode=LAN_MONITOR -A rapid_arp_count=5 -A poll_interval=0,500000
Addition of links 0, 1 to lan900 succeeded.
[root@ignite] / # nwmgr -s -S apa -A all --saved --from cu
[root@ignite] / #
```

To show the new config:

```
[root@ignite] ~ # nwmgr -g

Name/          Interface Station          Sub-   Interface      Related
ClassInstance  State     Address        system   Type           Interface
============== ========= ============== ======== ============== =========
lan2           DOWN      0x001E0B45E09A igelan   1000Base-SX
lan0           UP        0x001E0B45E09C igelan   1000Base-SX    lan900
lan1           UP        0x001E0B45E09D igelan   1000Base-SX    lan900
lan3           DOWN      0x001E0B45E09B igelan   1000Base-SX
lan900         UP        0x001E0B45E09C hp_apa   hp_apa
lan901         DOWN      0x000000000000 hp_apa   hp_apa
lan902         DOWN      0x000000000000 hp_apa   hp_apa
lan903         DOWN      0x000000000000 hp_apa   hp_apa
lan904         DOWN      0x000000000000 hp_apa   hp_apa
[root@ignite] ~ # netstat -ni
Name      Mtu  Network         Address         Ipkts              Ierrs Opkts              Oerrs Coll
lo0      32808 127.0.0.0       127.0.0.1       10100925           0     10100956           0     0
lan900    1500 10.31.4.0       10.31.4.37      256                0     92                 0     0
[root@ignite] ~ #
[root@ignite] ~ # nwmgr -g -S apa
Class    Mode        Load      Speed-               Membership
Instance             Balancing Duplex
======== =========== ========= ==================== ===========================
lan900   LAN_MONITOR LB_HS     1 Gbps Full Duplex   0,1
lan901   Not_Enabled LB_MAC    0 Mbps                -
lan902   Not_Enabled LB_MAC    0 Mbps                -
lan903   Not_Enabled LB_MAC    0 Mbps                -
lan904   Not_Enabled LB_MAC    0 Mbps                -
[root@ignite] ~ #
[root@ignite] ~ # nwmgr -S apa -I 900
Class    Parent APA         Mode        Load      Membership
Instance PPA    State                   Balancing
======== ====== =========== =========== ========= =============================
lan900    -     Up          LAN_MONITOR LB_HS     0,1
[root@ignite] ~ #
[root@ignite] ~ # nwmgr -g -S apa -I 900 -v
lan900 current values:
 Speed = 1 Gbps Full Duplex
 MTU = 1500
 Virtual Maximum Transmission Unit = 0
 MAC Address = 0x001e0b45e09c
 Network Management ID = 6
 Features = Linkagg Interface
 IPV4 Recv CKO
 IPV4 Send CKO
 VLAN Support
 VLAN Tag Offload
 64Bit MIB Support
 Load Distribution Algorithm = LB_HS
 Mode = LAN_MONITOR
 Parent PPA =  -
 APA State = Up
 Membership = 0,1
 Active Port(s) = 0
 Ready Port(s) = 1
 Not Ready Port(s) =  -
 Connected Port(s) = 1
 Polling Interval = 10000000
 Dead Count = 3
 Rapid ARP = on
 Rapid ARP Interval = 1.0 second(s)
 Rapid ARP Count = 10
 Failover Policy = PRIORITY_BASED
[root@ignite] ~ # ifconfig lan900
lan900: flags=1843<UP,BROADCAST,RUNNING,MULTICAST,CKO>
 inet 10.31.4.37 netmask ffffff00 broadcast 10.31.4.255
[root@ignite] ~ #
```

If you want to delete an existent failover group:

```
[root@ignite] ~ # nwmgr -d -S apa -A links=all -c lan900 --force
Deletion of links all ports from lan900 succeeded.
[root@ignite] ~ # nwmgr -s -S apa -A all --saved --from cu
[root@ignite] ~ #
```

Or if you just want to remove a interface from the group:

```
[root@ignite] ~ # nwmgr -d -S apa -c lan901 -A links=1
```

And we are finished, I still have 11.23 systems and I want APA for them too so I will post how to do it with the `lanadmin` command.

See you next time.

Juanma.
