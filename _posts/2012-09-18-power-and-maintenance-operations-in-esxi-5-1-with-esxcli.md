---
title: Power and maintenance operations in ESXi 5.1 with esxcli
date: 2012-09-18
type: post
classes: wide
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- esxcli
- ESXi
- sysadmin
- systems administration
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

**ESXi 5.1** comes with many improvements and one of them is new namespaces and commands in `esxcli`.

Those new commands enable a system administrator to perform a shutdown, a reboot or a maintenance operation in a host.

Under the `system` namespace the new commands are the equivalents of the classic `vicfg/esxcfg-hostops` which until now was the only way to perform such kind of operations with vCLI and are also accessible locally on ESXi Shell.

[![](/assets/images/esxcli_system_namespace.png "esxcli system namespace available commands")]({{site.url}}/assets/images/esxcli_system_namespace.png)

### Maintenance mode operations

Getting the basic usage of the command is as simple as always. You can
perform two operations.

-   Get the state of the host
-   Put the the host in or out of Maintenance Mode

```
~ # esxcli system maintenanceMode
Usage: esxcli system maintenanceMode {cmd} [cmd options]
Available Commands:
  get                   Get the maintenance mode state of the system.
  set                   Enable or disable the maintenance mode of the system.
~ #
```

##### Get the state of the host

```
~ # esxcli system maintenanceMode get
Disabled
~ #
```

##### Put the host in Maintenance Mode

```
~ # esxcli system maintenanceMode set -e true -t 0
~ #
~ # esxcli system maintenanceMode get
Enabled
~ #
```

### Power operations

With the `shutdown` command the host can be either rebooted or shutdown. If the ESXi server is not in Maintenance Mode mode the operation will not be allowed.

```
~ # esxcli system shutdown
Usage: esxcli system shutdown {cmd} [cmd options]
Available Commands:
  poweroff              Power off the system. The host must be in maintenance mode.
  reboot                Reboot the system. The host must be in maintenance mode.
~ #
```

For both task the delay and reason parameter must be provided.

```
~ # esxcli system shutdown poweroff
Error: Missing required parameter -r|--reason
Usage: esxcli system shutdown poweroff [cmd options]
Description:
  poweroff              Power off the system. The host must be in maintenance mode.
Cmd options:
  -d|--delay=<long>     Delay interval in seconds
  -r|--reason=<str>     Reason for performing the operation (required)
~ #
```

##### Power off the host

```
~ # esxcli system shutdown poweroff --delay=10 --reason=”Hardware maintenance”
```

##### Reboot the host

```
~ # esxcli system shutdown reboot -d 10 –r “Patches applied”
```

Juanma.
