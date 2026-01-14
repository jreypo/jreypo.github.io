---
title: Change ESXi 5.0 hostname with esxcli
date: 2011-11-15
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- esxcli
- ESXi Shell
- ESXi5
- sysadmin
- systems administration
- VMware
- vSphere
showComments: true
---

Yes another post about `esxcli`, what can I say I'm studying very hard for my VCP5 and from time to time this kind of unknown information, at least for me, arise and I believe it can be useful for some of you.

Again we are going to make use of the system namespace.

```text
~ # esxcli system hostname
Usage: esxcli system hostname {cmd} [cmd options]
Available Commands:
  get                   Get the host, domain or fully qualified name of the ESX host.
  set                   This command allows the user to set the hostname, domain name or fully qualified domain name of the ESX host.
~ #
```

First task of course is to get current hostname.

```text
~ # esxcli system hostname get
   Domain Name: vjlab.local
   Fully Qualified Domain Name: esxi5.vjlab.local
   Host Name: esxi5
~ #
```

Next change the hostname, but you should check before what options are at your disposal by getting the command help.

```text
~ # esxcli system hostname set --help
Usage: esxcli system hostname set [cmd options]
Description:
  set                   This command allows the user to set the hostname, domain name or fully qualified domain name of the ESX host.
Cmd options:
  -d|--domain=<str>     The domain name to set for the ESX host. This option is mutually exclusive with the --fqdn option.
  -f|--fqdn=<str>       Set the fully qualified domain name of the ESX host.
  -H|--host=<str>       The host name to set for the ESX host. This name should not contain the DNS domain name of the host and can only contain letters, numbers and '-'. NOTE this is not
                        the fully qualified name, that can be set with the --fqdn option. This option is mutually exclusive with the --fqdn option.
~ #
```

Interesting, you can change the short hostname, the domain or the fully qualified domain name. Take into account that `--fqdn` option is mutually exclusive with the others.

We are going to try all of them.

## Domain

```text
~ # esxcli system hostname set --domain=jreypo.local
~ #
~ # esxcli system hostname get
   Domain Name: jreypo.local
   Fully Qualified Domain Name: esxi5.jreypo.local
   Host Name: esxi5
~ #
```

## Short hostname

```text
    ~ # esxcli system hostname set --host=esxi5-2
    ~ #
    ~ # esxcli system hostname get
       Domain Name: jreypo.local
       Fully Qualified Domain Name: esxi5-2.jreypo.local
       Host Name: esxi5-2
    ~ #
```

## Fully qualified domain name

```text
~ # esxcli system hostname set --fqdn=esxi5.vjlab.local
~ #
~ # esxcli system hostname get
   Domain Name: vjlab.local
   Fully Qualified Domain Name: esxi5.vjlab.local
   Host Name: esxi5
~ #
```

Juanma.
