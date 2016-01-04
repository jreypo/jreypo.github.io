---
layout: post
title: FirewallD quickstart guide
date: 2015-04-08
type: post
published: true
status: publish
categories:
- Linux
- Red Hat
- Security
- Sysadmin
tags:
- CentOS
- Fedora
- firewall
- firewall-cmd
- firewalld
- Linux
- Red Hat
- RHEL
- RHEL7
- sysadmin
- systems administration
author: juan_manuel_rey
comments: true
---
**FirewallD**, or Dynamic Firewall Manager, is the replacement for the **IPTables** firewall in **Red Hat Enterprise Linux**. The main improvement over IPTables is the capacity to make changes without the need to restart the whole firewall service.

FirewallD was first introduced in Fedora 18 and has been the default firewall mechanism for Fedora since then. Finally this year Red Hat decided to include it in RHEL 7, and of course it also made its way to the different RHEL clones like CentOS 7 and Scientific Linux 7.

### Checking FirewallD service status

To get the basic status of the service simply use `firewall-cmd --state`.

{% highlight text %}
[root@centos7 ~]# firewall-cmd --state
running
[root@centos7 ~]#
{% endhighlight %}

If you need to get a more detailed state of the service you can always use `systemctl` command.

{% highlight text %}
[root@centos7 ~]# systemctl status firewalld.service
firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled)
   Active: active (running) since Wed 2014-11-19 06:47:42 EST; 32min ago
 Main PID: 873 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─873 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid

Nov 19 06:47:41 centos7.vlab.local systemd[1]: Starting firewalld - dynamic firewall daemon...
Nov 19 06:47:42 centos7.vlab.local systemd[1]: Started firewalld - dynamic firewall daemon.
[root@centos7 ~]#
{% endhighlight %}

To enable or disable FirewallD again use `systemctl` commands.

{% highlight text %}
systemctl enable firewalld.service
systemctl disable firewalld.service
{% endhighlight %}

### Managing firewall zones

FirewallD introduces the zones concept, a zone is no more than a way to define the level of trust for a set of connections. A connection definition can only be part of one zone at the same time but zones can be grouped. There is a set of predefined zones:

-   **Public** - For use in public areas. Only selected incoming connections are accepted.
-   **Drop** - Any incoming network packets are dropped, there is no reply. Only outgoing network connections are possible.
-   **Block** - Any incoming network connections are rejected with an icmp-host-prohibited message for IPv4 and icmp6-adm-prohibited for IPv6. Only network connections initiated within this system are possible.
-   **External** - For use on external networks with masquerading enabled especially for routers. Only selected incoming connections are accepted.
-   **DMZ** - For computers DMZ network, with limited access to the internal network. Only selected incoming connections are accepted.
-   **Work** - For use in work areas. Only selected incoming connections are accepted.
-   **Home** - For use in home areas. Only selected incoming connections are accepted.
-   **Trusted** - All network connections are accepted.
-   **Internal** - For use on internal networks. Only selected incoming connections are accepted.

By default all interfaces are assigned to the public zone. Each zone is defined in its own XML file stored in `/usr/lib/firewalld/zones`. For example the public zone XML file looks like this.

{% highlight text %}
root@centos7 zones]# cat public.xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
</zone>
[root@centos7 zones]#
{% endhighlight %}

Retrieve a simple list of the existing zones.

{% highlight text %}
[root@centos7 ~]# firewall-cmd --get-zones
block dmz drop external home internal public trusted work
[root@centos7 ~]#
{% endhighlight %}

Get a detailed list of the same zones.

{% highlight text %}
firewall-cmd --list-all-zones
{% endhighlight %}

Get the default zone.

{% highlight text %}
[root@centos7 ~]# firewall-cmd --get-default-zone
public
[root@centos7 ~]#
{% endhighlight %}

Get the active zones.

{% highlight text %}
[root@centos7 ~]# firewall-cmd --get-active-zones
public
  interfaces: eno16777736 virbr0
[root@centos7 ~]#
{% endhighlight %}

Get the details of a specific zone.

{% highlight text %}
[root@centos7 zones]# firewall-cmd --zone=public --list-all
public (default, active)
  interfaces: eno16777736 virbr0
  sources:
  services: dhcpv6-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:

[root@centos7 zones]#
{% endhighlight %}

Change the default zone.

{% highlight text %}
firewall-cmd --set-default-zone=home
{% endhighlight %}

### Interfaces and sources

Zones can be bound to a network interface and to a specific network addressing or source.

Assign an interface to a different zone, the first command assigns it temporarily and the second makes it permanently.

{% highlight text %}
firewall-cmd --zone=home --change-interface=eth0
firewall-cmd --permanent --zone=home --change-interface=eth0
{% endhighlight %}

Retrieve the zone an interface is assigned to.

{% highlight text %}
[root@centos7 zones]# firewall-cmd --get-zone-of-interface=eno16777736
public
[root@centos7 zones]#
{% endhighlight %}

Bound the zone `work` to a source.

{% highlight text %}
firewall-cmd --permanent --zone=work --add-source=192.168.100.0/27
{% endhighlight %}

List the sources assigned to a zone, in this case `work`.

{% highlight text %}
[root@centos7 ~]# firewall-cmd --permanent --zone=work --list-sources
172.16.10.0/24 192.168.100.0/27
[root@centos7 ~]#
{% endhighlight %}

### Services

FirewallD can assign services permanently to a zone, for example to assign `http` service to the `dmz` zone. A service can be also assigned to multiple zones.

{% highlight text %}
[root@centos7 ~]# firewall-cmd --permanent --zone=dmz --add-service=http
success
[root@centos7 ~]# firewall-cmd --reload
success
[root@centos7 ~]#
{% endhighlight %}

List the services assigned to a given zone.

{% highlight text %}
[root@centos7 ~]# firewall-cmd --list-services --zone=dmz
http ssh
[root@centos7 ~]#
{% endhighlight %}

### Other operations

Besides of Zones, interfaces and Services management FirewallD like other firewalls can perform several network related operations like masquerading, set direct rules and manage ports.

#### Masquerading and port forwarding

Add masquerading to a zone.

{% highlight text %}
firewall-cmd --zone=external --add-masquerade
{% endhighlight %}

Query if masquerading is enabled in a zone.

{% highlight text %}
[root@centos7 ~]# firewall-cmd --zone=external --query-masquerade
yes
[root@centos7 ~]#
{% endhighlight %}

You can also set port redirection. For example to forward traffic originally intended for port `80/tcp` to port `8080/tcp`.

{% highlight text %}
firewall-cmd --zone=external --add-forward-port=port=80:proto=tcp:toport=8080
{% endhighlight %}

A destination address can also bee added to the above command.

{% highlight text %}
firewall-cmd --zone=external --add-forward-port=port=80:proto=tcp:toport=8080:toaddr=172.16.10.21
{% endhighlight %}

#### Set direct rules

Create a firewall rule for `8080/tcp` port.

{% highlight text %}
firewall-cmd --direct --add-rule ipv4 filter INPUT -p tcp -m state --state NEW -m tcp --dport 8080 -j ACCEPT
{% endhighlight %}

#### Port management

Allow a port temporary in a zone.

{% highlight text %}
firewall-cmd --zone=dmz --add-port=8080/tcp
{% endhighlight %}

Hopefully you found the post useful to start working with FirewallD. Comments are welcome.

Juanma.
