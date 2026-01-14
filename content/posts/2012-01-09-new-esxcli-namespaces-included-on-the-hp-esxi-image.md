---
title: New esxcli namespaces included on the HP ESXi image
date: 2012-01-09
tags:
- esxi
- hp-servers
- scripting
- vmware
- vsphere

showComments: true
---

If your VMware infrastructure runs on top of HP servers, rack or blade, you should be familiar with the HP customized ESXi images. Hewlett Packard has been releasing theses images since ESXi 4.0 and with the release of vSphere 5 a new image has also been released.

This images comes bundled with utilities and drivers not included in the standard VMware image that allows ESXi to run flawlessly on HP hardware.

You can retrieve the packages specifically provided by using `esxcli` command.

```text
~ # esxcli software vib list |grep Hewlett-Packard
char-hpcru            5.0.0.8-1OEM.500.0.0.434156         Hewlett-Packard     PartnerSupported  2011-04-16 
char-hpilo            500.9.0.0.8-1OEM.500.0.0.434156     Hewlett-Packard     PartnerSupported  2011-04-16 
hp-esx-license        1.0-03                              Hewlett-Packard     PartnerSupported  2011-04-16 
hp-smx-provider       500.02.10.13.44-434156              Hewlett-Packard     VMwareAccepted    2011-04-16 
hpbootcfg             01-00.10                            Hewlett-Packard     PartnerSupported  2011-04-16 
hponcfg               03-02.04                            Hewlett-Packard     PartnerSupported  2011-04-16 
~ #
```

To get detailed information of any of those packages use the following command.

```text
~ # esxcli software vib get -n hp-smx-provider
Hewlett-Packard_bootbank_hp-smx-provider_500.02.10.13.44-434156
   Name: hp-smx-provider
   Version: 500.02.10.13.44-434156
   Type: bootbank
   Vendor: Hewlett-Packard
   Acceptance Level: VMwareAccepted
   Summary: HP Insight Management WBEM Providers for ESXi
   Description: HP Insight Management WBEM Providers for ESXi
   Release Date: 2011-07-15
   Depends:
   Conflicts:
   Replaces:
   Provides: cim.DMTF.DSP1004 = 1.0.0, cim.SNIA.FC-HBA = 1.3.0
   Maintenance Mode Required: False
   Hardware Platforms Required: HP, hp, Hewlett-Packard, Hewlett-Packard Company
   Live Install Allowed: True
   Live Remove Allowed: False
   Stateless Ready: True
   Overlay: False
   Tags:
   Payloads: hp-smx-provider
~ #
```

Aside from the drivers and monitoring agents, the HP image also adds two additional namespaces to `esxcli`.

- `hp`
- `hpbootcfg`

The first one, hp, manage the HP NMI driver.

```text
~ # esxcli hp
Usage: esxcli hp {cmd} [cmd options]
Available Namespaces:
  hpnmi                 The default set of operations for the hpnmi command
~ # esxcli hp hpnmi
Usage: esxcli hp hpnmi {cmd} [cmd options]
Available Commands:
  load                  Verifies HP server and Loads hpnmi module on supported servers
~ #
```

The `hpbootcfg` namespace is used to configure the device boot order of the server. This can be permanent or a one time boot change.

```text
~ # esxcli hpbootcfg
Usage: esxcli hpbootcfg {cmd} [cmd options]
Available Commands:
 execute execute hpbootcfg command with options parameter
 help show hpbootcfg help
 show show current hpbootcfg settings
~ #
~ # esxcli hpbootcfg show
00 00: Normal Device first, normal boot process

~ #
~ # esxcli hpbootcfg execute --help
Usage: esxcli hpbootcfg execute [cmd options]
Description:
 execute execute hpbootcfg command with options parameter
Cmd options:
 -b|--bypass bypass F1/F2
 -C|--cdrom CD ROM first
 -D|--defaults Set Defaults everywhere
 -d|--dialout one time remote dial out
 -F|--floppy Floppy first
 -H|--harddrive Harddrive first
 -n|--network one time remote network
 -P|--pxe one time boot to PXE
 -Q|--qcu one time boot to quick configuration utility
 -R|--rbsu one time boot to RBSU
 -r|--remote one time remote
 -S|--scu one time boot to system configuration utility
 -T|--tape Tape first
~ #
```

Juanma.
