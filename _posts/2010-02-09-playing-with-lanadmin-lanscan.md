---
title: Playing with lanadmin & lanscan
date: 2010-02-09
type: post
classes: wide
published: true
status: publish
categories:
- HP-UX
- Networking
- Sysadmin
tags:
- 11iv1
- 11iv2
- APA
- commands
- HP-UX
- lanadmin
- lanscan
- networking
- sysadmin
- systems administration
author: juan_manuel_rey
comments: true
---

Current release of HP-UX, 11.31, has the handy `nwmgr` to handle networking tasks, but for years instead of `nwmgr` we've been playing with `lanscan` and `lanadmin`, `linkloop` as well, to perform many networking tasks on the 11.23 release and previous ones. And surely some of you, just like myself, still have 11iv2 systems up and running. Following is a small list of tips and tasks for `lanadmin` from one of my *how-to* files.

## Lanscan

`lanscan` is used to get information about the LAN interfaces.

```text
root@sap01:~# lanscan
Hardware Station        Crd Hdw   Net-Interface  NM  MAC       HP-DLPI DLPI
Path     Address        In# State NamePPA        ID  Type      Support Mjr#
0/0/0/1/0 0x001A4B07F002 0   UP    lan0 snap0     1   ETHER     Yes     119
0/0/0/1/1 0x001A4B07F003 1   UP    lan1 snap1     2   ETHER     Yes     119
0/0/1/1/0 0x0018FE2D7EE7 2   UP    lan2 snap2     3   ETHER     Yes     119
0/0/9/1/0 0x0018FE2D7EF4 3   UP    lan3 snap3     4   ETHER     Yes     119
0/0/10/1/0 0x000CFC0046B9 4   UP    lan4 snap4     5   ETHER     Yes     119
0/0/12/1/0 0x000CFC004672 5   UP    lan5 snap5     6   ETHER     Yes     119
LinkAgg0 0x000000000000 900 DOWN  lan900 snap900 9   ETHER     Yes     119
LinkAgg1 0x000000000000 901 DOWN  lan901 snap901 10  ETHER     Yes     119
LinkAgg2 0x000000000000 902 DOWN  lan902 snap902 11  ETHER     Yes     119
LinkAgg3 0x000000000000 903 DOWN  lan903 snap903 12  ETHER     Yes     119
LinkAgg4 0x000000000000 904 DOWN  lan904 snap904 13  ETHER     Yes     119
root@sap01:~#
```

A verbose version can be obtained with the `-v` switch, but for me this switch has a glitch since you can't query for a single LAN card:

```text
root@sap01:~# lanscan -v
-------------------------------------------------------------------------------
Hardware Station        Crd Hdw   Net-Interface  NM  MAC       HP-DLPI DLPI
Path     Address        In# State NamePPA        ID  Type      Support Mjr#
0/0/0/1/0 0x001A4B07F002 0   UP    lan0 snap0     1   ETHER     Yes     119

Extended Station                           LLC Encapsulation
Address                                    Methods
0x001A4B07F002                             IEEE HPEXTIEEE SNAP ETHER NOVELL 

Driver Specific Information
iether
-------------------------------------------------------------------------------
...
root@sap01:~#
```

There are other options for `lanscan` that can be used to obtain more simple info in a script friendly list format:

```text
root@sap01:~# lanscan -a
0x001A4B07F002
0x001A4B07F003
0x0018FE2D7EE7
0x0018FE2D7EF4
0x000CFC0046B9
0x000CFC004672
0x000000000000
0x000000000000
0x000000000000
0x000000000000
0x000000000000
root@sap01:~# lanscan -i
lan0 snap0
lan1 snap1
lan2 snap2
lan3 snap3
lan4 snap4
lan5 snap5
lan900 snap900
lan901 snap901
lan902 snap902
lan903 snap903
lan904 snap904
root@sap01:~#
```

## Lanadmin  

`lanadmin` command, according to its man page, allows you to:

- Display and change the station address.
- Display and change the 802.5 Source Routing options (RIF).
- Display and change the maximum transmission unit (MTU).
- Display and change the speed setting.
- Clear the network statistics registers to zero.
- Display the interface statistics.
- Display the interface usage information.
- Reset the interface card, thus executing its self-test.
- Configure VLANs on the cards that support VLAN.

It can be used in two ways, if invoked with no options from the shell it will present a menu style interface where different tasks can be performed. Following is am example to illustrate.

```text
          LOCAL AREA NETWORK ONLINE ADMINISTRATION, Version 1.0
                       Tue , Feb 9,2010  14:22:27

               Copyright 1994 Hewlett Packard Company.
                       All rights are reserved.

Test Selection mode.

        lan      = LAN Interface Administration
        menu     = Display this menu
        quit     = Terminate the Administration
        terse    = Do not display command menu
        verbose  = Display command menu

Enter command: lan

LAN Interface test mode. LAN Interface PPA Number = 0

        clear    = Clear statistics registers
        display  = Display LAN Interface status and statistics registers
        end      = End LAN Interface Administration, return to Test Selection
        menu     = Display this menu
        ppa      = PPA Number of the LAN Interface
        quit     = Terminate the Administration, return to shell
        reset    = Reset LAN Interface to execute its selftest
        specific = Go to Driver specific menu

Enter command: display

                      LAN INTERFACE STATUS DISPLAY
                       Tue , Feb 9,2010  14:22:31

PPA Number                      = 0
Description                     = lan0 HP PCI Core I/O 1000Base-T Release B.11.23.0712.01
Type (value)                    = ethernet-csmacd(6)
MTU Size                        = 1500
Speed                           = 1000000000
Station Address                 = 0x14c2650091
Administration Status (value)   = up(1)
Operation Status (value)        = up(1)
Last Change                     = 419
Inbound Octets                  = 120454615
Inbound Unicast Packets         = 863761
Inbound Non-Unicast Packets     = 4327
Inbound Discards                = 0
Inbound Errors                  = 0
Inbound Unknown Protocols       = 12
Outbound Octets                 = 145033817
Outbound Unicast Packets        = 1285500
Outbound Non-Unicast Packets    = 221
Outbound Discards               = 0
Outbound Errors                 = 0
Outbound Queue Length           = 0
Specific                        = 655367

Press  to continue

Ethernet-like Statistics Group

Index                           = 1
Alignment Errors                = 0
FCS Errors                      = 0
Single Collision Frames         = 0
Multiple Collision Frames       = 0
Deferred Transmissions          = 0
Late Collisions                 = 0
Excessive Collisions            = 0
Internal MAC Transmit Errors    = 0
Carrier Sense Errors            = 0
Frames Too Long                 = 0
Internal MAC Receive Errors     = 0

LAN Interface test mode. LAN Interface PPA Number = 0

        clear    = Clear statistics registers
        display  = Display LAN Interface status and statistics registers
        end      = End LAN Interface Administration, return to Test Selection
        menu     = Display this menu
        ppa      = PPA Number of the LAN Interface
        quit     = Terminate the Administration, return to shell
        reset    = Reset LAN Interface to execute its selftest
        specific = Go to Driver specific menu

Enter command:
```

When used with options from the command line `lanadmin` can perform the same tasks as as in the menu interface on each LAN card. Here are some of the most common features I've been using for years.

- Display interface info:

```text
root@sap01:~# lanadmin -x card_info  1
*********** Version Information **********
Driver version: B.11.23.0712
Firmware version: N/A
Chip version: 0x3
PCI Sub-System ID: 0x12a6
PCI Sub-Vendor ID: 0x103c
Board Revision: D4503807
Software Key: 0
Engineering Date Code: A-4731

********** Card Setting ***********
Driver State: IETHER_ONLINE
Auto Negotiation: On
Flow Control: On
Send Max Buf Descriptors: 1
Recv Max Buf Descriptors: 1
Send Coalesced Ticks: 150
Recv Coalesced Ticks: 0
root@sap01:~#
```

- Display Auto-Port Aggregation status:

```text
root@sap01:~# lanadmin -x -v 900
Link Aggregate PPA #      : 900
Number of Ports           : 2
Ports PPA                 : 0 1
Link Aggregation State    : LINKAGG MANUAL
Load Balance Mode         : Hot Standby (LB_HOT_STANDBY)

root@sap01:~#
```

- Show speed settings:

```text
root@sap01:/# lanadmin -x 1
Speed = 1000 Full-Duplex.
Autonegotiation = On.

root@sap01:/#
```

- Creating an Aggregation link:

```text
roo@sap01:/# lanadmin -X -a 1 2 900
```

- Show load balancing algorithm in APA:

```text
root@sap02:/etc# lanadmin -x -l 900
Load Balancing = Hot Standby (LB_HOT_STANDBY)
root@sap02:/etc#
```

- Display MAC address:

```text
root@sap01:~# lanadmin -a 1  
Station Address                 = 0x001a4b07f003
root@sap01:~#
```

- Display driver and adapter statistics:

```text
root@sap01:/# lanadmin -x stats drv 1
****** Driver Statistics ******        
In Packet Error                                        0
Out Packet Error                                       0
Loopback packets                                      44
Link Down events                                       0

****** Host Command Statistics ******  
nicCmdsDelMCastAddr                                    0
nicCmdsSetMACAddr                                      0
nicCmdsSetPromiscMode                                  0
nicCmdsSetMulticastMode                                0
nicCmdsClearStats                                      1

****** NIC Events Statistics ******    
nicEventsFirmwareOperational                           0
nicEventsStatsUpdated                                  0
nicEventsLinkStateChanged                              1
nicEventsMCastListUpdated                              0

****** Interface Statistics ******     
ifIndex                                                2
ifType                                                 6
ifMtu                                               1500
ifSpeed                                       1000000000
ifAdminStatus                                          1
ifOperStatus                                           1
ifLastChange                                          36
ifInDiscards                                           0
ifInErrors                                             0
ifInUnknownProtos                                  87777
ifOutDiscards                                          0
ifOutErrors                                            0
ifOutQLen                                              0
ifInOctets_low                                1205643914
ifInOctets_high                                        0
ifInUcastPkts_low                                8695826
ifInUcastPkts_high                                     0
ifInMulticastPkts_low                                  0
ifInMulticastPkts_high                                 0
ifInBroadcastPkts_low                              87777
ifInBroadcastPkts_high                                 0
ifOutOctets_low                               1200310422
ifOutOctets_high                                       0
ifOutUcastPkts_low                               8696015
ifOutUcastPkts_high                                    0
ifOutMulticastPkts_low                                 0
ifOutMulticastPkts_high                                0
ifOutBroadcastPkts_low                                 0
ifOutBroadcastPkts_high                                0
root@sap01:/#
```

- Show Vital Product Data, a really funny name ;-) don't you think?

```text
root@sap01:/# lanadmin -x vpd 0
*********** Vital Product Data **********
Product Description: PCI/PCI-X 10/100/1000BT Dual Ethernet Adapter
Part Number: A7012-60601
Engineering Date Code: A-4731
Part Serial Number: 001A4B07F002
Misc. Information: 7.5W
Mfd. Date: 4749
Checksum: 0xb
EFI Version: 03048
ROM Firmware Version: N/A
Asset Tag: N/A
root@sap01:/#
```

- Show card type:

```text
root@sap01:~# lanadmin -x type 1     
1000Base-T

root@sap01:~#
```

And we are finished. Probably I'm forgetting a more interesting uses of `lanadmin` if you have other everyday use please comment :-)

Juanma.
