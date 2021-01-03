---
title: More HP sauce on ESXi 5.0
date: 2012-02-24
type: post
classes: wide
published: true
status: publish
categories:
- HP
- Virtualization
- VMware
tags:
- ESXi5
- HP
- HP ESXi
- hponcfg
- ProLiant
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

On my [first post about HP ESXi 5.0](/2012/01/09/new-esxcli-namespaces-included-on-the-hp-esxi-image/ "New esxcli namespaces included on the HP ESXi image") customized image I discussed about the new `esxcli` namespaces added by HP. But those tools aren't the only ones included.

If you list the software bundles and filter the output to display only the included by HP will notice an `hponcfg` package.

```
~ # esxcli software vib list |grep Hewlett
char-hpcru            5.0.0.8-1OEM.500.0.0.434156         Hewlett-Packard     PartnerSupported  2011-05-24 
char-hpilo            500.9.0.0.8-1OEM.500.0.0.434156     Hewlett-Packard     PartnerSupported  2011-05-24 
hp-smx-provider       500.02.10.13.44-434156              Hewlett-Packard     VMwareAccepted    2011-05-24 
hpbootcfg             01-00.10                            Hewlett-Packard     PartnerSupported  2011-05-24 
hponcfg               03-02.04                            Hewlett-Packard     PartnerSupported  2011-05-24 
~ #
```

`hponcfg`, also included on ProLiant Support Pack for Linux, is a tool that enable a System Administrator to manage the iLO of a ProLiant server from the operative system.

```
~ # esxcli software vib get -n hponcfg
Hewlett-Packard_bootbank_hponcfg_03-02.04
   Name: hponcfg
   Version: 03-02.04
   Type: bootbank
   Vendor: Hewlett-Packard
   Acceptance Level: PartnerSupported
   Summary: HP ProLiant Lights-Out Configuration Utility for ESXi
   Description: HP ProLiant Lights-Out Configuration Utility for ESXi
   Release Date: 2011-08-09
   Depends:
   Conflicts:
   Replaces:
   Provides:
   Maintenance Mode Required: False
   Hardware Platforms Required: HP, Hewlett-Packard Company, Hewlett-Packard, hp
   Live Install Allowed: False
   Live Remove Allowed: False
   Stateless Ready: True
   Overlay: False
   Tags:
   Payloads: hponcfg
~ #
```

The tool is located at `/opt/hp/tools`.

```
/opt/hp/tools # ls
hpbootcfg         hpbootcfg_esxcli  hponcfg
/opt/hp/tools #
```

Launch the utility without arguments and you will get the usage and it will also display current firmware version of the iLO.

```
/opt/hp/tools # ./hponcfg
HP Lights-Out Online Configuration utility
Version 3.2-4 (c) Hewlett-Packard Company, 2011
Firmware Revision = 1.26 Device type = iLO 3 Driver name = hpilo
USAGE:
  hponcfg  -?
  hponcfg  -h
  hponcfg  -m minFw
  hponcfg  -r [-m minFw ]
  hponcfg  [-a] -w filename [-m minFw]
  hponcfg  -g [-m minFw]
  hponcfg  -f filename [-l filename] [-s namevaluepair] [-v] [-m minFw]
  hponcfg  -i [-l filename] [-s namevaluepair] [-v] [-m minFw]
  -h,  --help           Display this message
  -?                    Display this message
  -r,  --reset          Reset the Management Processor to factory defaults
  -f,  --file           Get/Set Management Processor configuration from "filename"
  -i,  --input          Get/Set Management Processor configuration from the XML input
                        received through the standard input stream.
  -w,  --writeconfig    Write the Management Processor configuration to "filename"
  -a,  --all            Capture complete Management Processor configuration to the file.
                        This should be used along with '-w' option
  -l,  --log            Log replies to "filename"
  -v,  --xmlverbose     Display all the responses from Management Processor
  -s,  --substitute     Substitute variables present in input config file
                        with values specified in "namevaluepairs"
  -g,  --get_hostinfo   Get the Host information
  -m,  --minfwlevel     Minimum firmware level
/opt/hp/tools #
```

As a non intrusive example you can use `-g` switch to get the server info.

```
/opt/hp/tools # ./hponcfg  -g
HP Lights-Out Online Configuration utility
Version 3.2-4 (c) Hewlett-Packard Company, 2011
Firmware Revision = 1.26 Device type = iLO 3 Driver name = hpilo
Host Information:
                        Server Name: esxi01.hp.local
                        Server Number:
/opt/hp/tools #
```

I'll let you to investigate the rest of the options carefully.

Juanma.
