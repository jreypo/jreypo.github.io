---
layout: post
title: Power and maintenance operations in ESXi 5.1 with esxcli
date: 2012-09-18
type: post
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- esxcli
- ESXi
- esxi 5.1
- sysadmin
- systems administration
- VMware
- vSphere
- vSphere 5.1
author: juan_manuel_rey
comments: true
---

**ESXi 5.1** comes with many improvements and one of them is new namespaces and commands in `esxcli`.

Those new commands enable a system administrator to perform a shutdown, a reboot or a maintenance operation in a host.

Under the `system` namespace the new commands are the equivalents of the classic `vicfg/esxcfg-hostops` which until now was the only way to perform such kind of operations with vCLI and are also accessible locally on ESXi Shell.

[![](/images/esxcli_system_namespace.png "esxcli system namespace available commands")]({{site.url}}/images/esxcli_system_namespace.png)

### Maintenance mode operations

Getting the basic usage of the command is as simple as always. You can
perform two operations.

-   Get the state of the host
-   Put the the host in or out of Maintenance Mode

{% highlight text %}
~ # esxcli system maintenanceMode
Usage: esxcli system maintenanceMode {cmd} [cmd options]
Available Commands:
  get                   Get the maintenance mode state of the system.
  set                   Enable or disable the maintenance mode of the system.
~ #
{% endhighlight %}

##### Get the state of the host

{% highlight text %}
~ # esxcli system maintenanceMode get
Disabled
~ #
{% endhighlight %}

##### Put the host in Maintenance Mode

{% highlight text %}
~ # esxcli system maintenanceMode set -e true -t 0
~ #
~ # esxcli system maintenanceMode get
Enabled
~ #
{% endhighlight %}

### Power operations

With the `shutdown` command the host can be either rebooted or shutdown. If the ESXi server is not in Maintenance Mode mode the operation will not be allowed.

{% highlight text %}
~ # esxcli system shutdown
Usage: esxcli system shutdown {cmd} [cmd options]
Available Commands:
  poweroff              Power off the system. The host must be in maintenance mode.
  reboot                Reboot the system. The host must be in maintenance mode.
~ #
{% endhighlight %}

For both task the delay and reason parameter must be provided.

{% highlight text %}
~ # esxcli system shutdown poweroff
Error: Missing required parameter -r|--reason
Usage: esxcli system shutdown poweroff [cmd options]
Description:
  poweroff              Power off the system. The host must be in maintenance mode.
Cmd options:
  -d|--delay=<long>     Delay interval in seconds
  -r|--reason=<str>     Reason for performing the operation (required)
~ #
{% endhighlight %}

##### Power off the host

{% highlight text %}
~ # esxcli system shutdown poweroff --delay=10 --reason=”Hardware maintenance”
{% endhighlight %}

##### Reboot the host

{% highlight text %}
~ # esxcli system shutdown reboot -d 10 –r “Patches applied”
{% endhighlight %}

Juanma.
