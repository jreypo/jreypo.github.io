---
layout: post
title: Quick tip - Troubleshooting networkd
date: 2016-02-08
type: post
published: true
status: publish
categories:
- Cloud-Native
- DevOps
- VMware
- Sysadmin
- Linux
tags:
- Cloud
- cloud-native
- Photon
- Linux
- sysadmin
- systems management
- systemd
- networkd
- devops
author: juan_manuel_rey
comments: true
---

To troubleshoot `networkd` in case of failure we can enable debug mode using the following procedure, this is applicable to Photon OS or any other linux system using `networkd`

First create `/etc/systemd/system/systemd-networkd.service.d/` directory.

```
mkdir -p /etc/systemd/system/systemd-networkd.service.d/
```

In the previous path create the `10-debug.conf` drop-in snippet with the following content.

```
[Service]
Environment=SYSTEMD_LOG_LEVEL=debug
```

Reload `systemd` and restart `systemd-networkd`.

```
systemctl daemon-reload
systemctl restart systemd-networkd
```

Using `journalctl` we can now see more detailed information in `networkd` log.

```
root@lightwave01 [ ~ ]# journalctl -u systemd-networkd -f
-- Logs begin at Sat 2016-02-06 20:42:47 UTC. --
Feb 07 03:01:07 lightwave01 systemd-networkd[428]: ICMPv6 CLIENT: Error sending Router Solicitation
Feb 07 03:01:10 lightwave01 systemd-networkd[428]: DHCP CLIENT (0xd25e35e9): DISCOVER
Feb 07 03:01:12 lightwave01 systemd-networkd[428]: ICMPv6 CLIENT: Error sending Router Solicitation
Feb 07 03:01:16 lightwave01 systemd-networkd[428]: DHCPv6 CLIENT: Next retransmission in 1.123266s
Feb 07 03:01:17 lightwave01 systemd-networkd[428]: DHCPv6 CLIENT: Next retransmission in 2.323319s
Feb 07 03:01:18 lightwave01 systemd-networkd[428]: DHCP CLIENT (0xd25e35e9): DISCOVER
Feb 07 03:01:19 lightwave01 systemd-networkd[428]: DHCPv6 CLIENT: Next retransmission in 4.642094s
Feb 07 03:01:24 lightwave01 systemd-networkd[428]: DHCPv6 CLIENT: Next retransmission in 9.555344s
Feb 07 03:01:33 lightwave01 systemd-networkd[428]: DHCP CLIENT (0xd25e35e9): DISCOVER
Feb 07 03:01:33 lightwave01 systemd-networkd[428]: DHCPv6 CLIENT: Next retransmission in 19.683553s
```

-- Juanma
