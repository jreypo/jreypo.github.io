---
layout: post
title: Network configuration in Photon OS
date: 2016-02-08
type: post
published: true
status: publish
categories:
- Cloud-Native
- DevOps
- VMware
- Sysadmin
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

Photon instances are usually meant to be configured automatically with Cloud-config, this includes hostname, network settings, additional packages, etc. However there are some corner cases where you may need to manually configure an static IP address, like a [Lightwave domain controller]({% post_url 2016-01-11-vmware-lightwave-multi-node-domain-setup %}) or a DNS server.

## Static addressing configuration

By default in Photon OS DHCP comes enabled for all present network interfaces. This is more than enough for testing however for more permanent services like a Lightwave server is desirable to use static IP addresses.

If you are used to work with Red Hat or any of its variants your first move will be to look at `/etc/sysconfig/network-scripts/` however there is nothing there, actually the path does not even exists. In Photon the network subsystem is controlled by `systemd-networkd`.

{% highlight text %}
root@lightwave02 [ /etc/systemd/network ]# cat 10-dhcp-en.network
[Match]
Name=*

[Network]
DHCP=yes
root@lightwave02 [ /etc/systemd/network ]#
{% endhighlight %}

Drop a unit file in `/etc/systemd/network` to configure static IP address for a certain interface. Remove the unit file used for DHCP, name the file `10-static-en.network` and include the following content.

{% highlight text %}
[Match]
Name=eno1

[Network]
Address=192.168.161.21/24
Gateway=192.168.161.2
DNS=192.168.161.9
{% endhighlight %}

The `[Match]` section defines the network interfaces the configruation will be applied to, in this case `eno1`, and the `[Network]` section the configuration itself.

To apply the new settings restart `systemd-networkd` service.

{% highlight text %}
systemctl restart systemd-networkd
{% endhighlight %}

## Multi-protocol configuration

If you need to configure static IP for one interface and leave DHCP for the rest create two unit files:

- `10-static-en.network`
- `20-dhcp-en.network`

In the first one include a configuration similar to the one described before and for the second one use the following DHCP unit file.

{% highlight text %}
[Match]
Name=en*

[Network]
DHCP=yes˚
{% endhighlight %}

## Static routes

To configure static routes, in the file `10-static-en.network` add any additional static routing information under the `[Route]` section.

{% highlight text %}
[Route]
Gateway=192.168.161.10
Destination=172.16.10.0/24
{% endhighlight %}

## Bond interfaces

Using `networkd` unit file we can create and configure a `bond` interface to provide load balancing and redundancy to the network connectivity within our Photon OS instance.

First remove any DHCP related unit file. And create three new files:

- `10-en.network`
- `20-bond.netdev`
- `30-bond-static.network`

`10-en.network` will indicart `networkd` to put our `eno` interfaces under `bond0`.

{% highlight text %}
[Match]
Name=eno*

[Network]
Bond=bond0
{% endhighlight %}

`20-bond.netdev` defines the behavior of the `bond0` interface.

{% highlight text %}
[NetDev]
Name=bond0
Kind=bond

[Bond]
Mode=balance-rr
MIIMonitorSec=10s
{% endhighlight %}

The section `[NetDev]` defines the name and kind of the network device and `[Bond]` defines de mode, by default `balance-rr` and `MIIMonitorSec` the interval of seconds for the link monitoring operation.

Finally `30-bond-static.network` stores the IP configuration for `bond0`.

{% highlight text %}
[Match]
Name=bond0

[Network]
Address=192.168.161.22/24
Gateway=192.168.161.2
DNS=192.168.161.6
{% endhighlight %}

I recommend to review `systemd-networkd` [official documentation](https://www.freedesktop.org/software/systemd/man/systemd-networkd.service.html) to get more details about the different options available. Stay tuned for more Photon content in future posts. And of course comments are welcome.

-- Juanma
