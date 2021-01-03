---
title: EMC Symmetrix Timefinder survival guide
date: 2010-12-10
type: post
classes: wide
published: true
status: publish
categories:
- EMC
- Storage
tags:
- BCV
- EMC
- EMC Timefinder
- Storage
- symcli
- Symmetrix
author: juan_manuel_rey
comments: true
---

During a previous project I had the opportunity to work very closely with the **EMC** people and Symmetrix arrays, in fact I got a couple of very good friends from that project. At the time I created a bunch of text files for my self  reference about EMC SRDF and Timefinder technologies.

Today I decided to review that files, give them some order, well sort of, and put them here as a survival guide/quick reference in the hope that will be of help to any of you. The first of this guides will be about **EMC Symmetrix Timefinder.**

I don't have sample output for every command, been more than a year since the last time I work with Timefinder, to complement my own samples I got several outputs from the Timefinder manuals.

**This is not a complete Timefinder usage guide**, just my personal notes taken from my direct experience with product.

### Timefinder Basics

EMC Timefinder is a replication solution that creates full volume copies. For the full-HP guys out there this is very similar to the XP or EVA Business Copy product.

There are two basic types of replication:

-   TimefinderClone - Creates point-in-time copies.
-   Timefinder/Snap - Creates pointer-based replicas, snapshots, only the changed data is written.

There are several optional components.

-   Timefinder/Mirror.
-   Timefinder/CG (Consistency Groups)
-   Timefinder/EIM (Exchange Integration Modules)
-   Timefinder/SIM (SQL Integration Modules)

Timefinder allows to retain multiple copies at different checkpoints for lowered RPO and RTO.

### Symcli basics

Following is a list of the most basic `symcli` commands necessary to get your way around when you perform any Symmetrix task, including Timefinder.

#### Get the list of the Symmetrix devices

```
root:/# symdev list
Symmetrix ID: 00029xxxxxxx
        Device Name           Directors                  Device
--------------------------- ------------- -------------------------------------
                                                                           Cap
Sym  Physical               SA :P DA :IT  Config        Attribute Sts      (MB)
--------------------------- ------------- -------------------------------------
0000 Not Visible            ???:? 01A:C0  BCV           Asst'd    RW       8632
0001 Not Visible            ???:? 16C:D0  BCV           Asst'd    RW       8632
0002 Not Visible            ???:? 01B:D0  BCV           Asst'd    RW       8632
0003 Not Visible            ???:? 16D:C0  BCV           Asst'd    RW       8632
…………………………………………………………………………………………………………………………………………………………………………………………………………………
0048 Not Visible            ???:? ???:??  VDEV          N/Grp'd   RW       8632
0049 Not Visible            ???:? ???:??  VDEV          N/Grp'd   RW       8632
004A Not Visible            ???:? ???:??  VDEV          N/Grp'd   RW       8632
004B Not Visible            ???:? ???:??  VDEV          N/Grp'd   NR       8632
004C Not Visible            ???:? ???:??  VDEV          N/Grp'd   NR       8632
004D Not Visible            ???:? ???:??  VDEV          N/Grp'd   NR       8632
004E Not Visible            ???:? ???:??  VDEV          N/Grp'd   NR       8632
004F Not Visible            ???:? ???:??  VDEV          N/Grp'd   NR       8632
0050 Not Visible            ???:? ???:??  VDEV          N/Grp'd   NR       8632
0051 Not Visible            ???:? ???:??  VDEV          N/Grp'd   NR       8632
0052 Not Visible            ???:? 16B:D1  2-Way Mir     N/A (SV)  RW       8632
0053 Not Visible            ???:? 01C:C0  2-Way Mir     N/A (SV)  RW       8632
0054 Not Visible            ???:? 16B:C0  2-Way Mir     N/A (SV)  RW       8632
…………………………………………………………………………………………………………………………………………………………………………………………………………………
```

#### List all available devices from a device group

```
root:/# symld -g dg_oradev_01 list
```

#### List host physical devices

```
root:/# sympd list
```

#### List the disk groups:

```
root:/# /usr/symcli/bin/symdg list

                       D E V I C E      G R O U P S

                                                          Number of
 Name               Type     Valid  Symmetrix ID  Devs   GKs  BCVs  VDEVs

 dg_oracle_prod1    REGULAR  Yes    00029xxxxxxx    26     0    26      0
 dg_oracle_prod2    REGULAR  Yes    00029xxxxxxx    21     0    21      0
 dg_rac_01          RDF1     Yes    00029xxxxxxx    23     0    23      0
 dg_clvx_01         RDF1     Yes    00029xxxxxxx     5     0     5      0
 dg_oradev_01       REGULAR  Yes    00029xxxxxxx     3     0     0      0
 dg_timetest_02     RDF1     Yes    00029xxxxxxx    16     0    16      0
 grupo1             RDF1     Yes    00029xxxxxxx    22     0     0      0

root:/#
```

#### Add devices to a disk group

-   Add physical devices

```
root:/# symld -g dg_oradev_01 add pd /dev/dsk/c2t4d12
```

-   Add Symmetrix devices

```
root:/# symld -g dg_oradev_01 add 006E
```

#### Get diskgroup detailed info.

```
root:/# /usr/symcli/bin/symdg show dg_prod_01

Group Name:  dg_prod_01

    Group Type                                   : RDF1     (RDFA)
    Device Group in GNS                          : No
    Valid                                        : Yes
    Symmetrix ID                                 : 00029xxxxxxx
    Group Creation Time                          : Mon Nov 29 18:49:29 2007
    Vendor ID                                    : EMC Corp
    Application ID                               : ECC

    Number of STD Devices in Group               :    2
    Number of Associated GK's                    :    0
    Number of Locally-associated BCV's           :    2
    Number of Locally-associated VDEV's          :    0
    Number of Remotely-associated VDEV's(STD RDF):    0
    Number of Remotely-associated BCV's (STD RDF):    0
    Number of Remotely-associated BCV's (BCV RDF):    0
    Number of Remotely-assoc'd RBCV's (RBCV RDF) :    0

    Standard (STD) Devices (2):
        {
        --------------------------------------------------------------------
        Sym               Cap
        LdevName              PdevName                Dev  Att. Sts     (MB)
        --------------------------------------------------------------------
        DEV001                N/A                     01C8      RW      8714
        DEV002                N/A                     01C9      RW      8714
        }

    BCV Devices Locally-associated (2):
        {
        --------------------------------------------------------------------
        Sym               Cap
        LdevName              PdevName                Dev  Att. Sts     (MB)
        --------------------------------------------------------------------
        BCV001                N/A                     08A8      RW      8714
        BCV002                N/A                     08A9      RW      8714
        }

    Device Group RDF Information
        {
        RDF Type                               : R1
        RDF (RA) Group Number                  : 2               (01)

        Remote Symmetrix ID                    : 000287xxxxxx

        R2 Device Is Larger Than The R1 Device : False

        RDF Pair Configuration                 : Normal
        RDF STAR Mode                          : False

        RDF Mode                               : Synchronous
        RDF Adaptive Copy                      : Disabled
        RDF Adaptive Copy Write Pending State  : N/A
        RDF Adaptive Copy Skew (Tracks)        : 32767

        RDF Device Domino                      : Disabled

        RDF Link Configuration                 : Fibre
        RDF Link Domino                        : Disabled
        Prevent Automatic RDF Link Recovery    : Disabled
        Prevent RAs Online Upon Power ON       : Enabled

        Device RDF Status                      : Ready           (RW)

        Device RA Status                       : Ready           (RW)
        Device Link Status                     : Ready           (RW)

        Device Suspend State                   : N/A
        Device Consistency State               : Disabled
        RDF R2 Not Ready If Invalid            : Disabled

        Device RDF State                       : Ready           (RW)
        Remote Device RDF State                : Write Disabled  (WD)

        RDF Pair State (  R1 <===> R2 )        : Synchronized

        Number of R1 Invalid Tracks            : 0
        Number of R2 Invalid Tracks            : 0

        RDFA Information:
            {
            Session Number                     : 1
            Cycle Number                       : 0
            Number of Devices in the Session   : 491
            Session Status                     : Inactive

            Session Consistency State          : N/A
            Minimum Cycle Time                 : 00:00:30
            Average Cycle Time                 : 00:00:00
            Duration of Last cycle             : 00:00:00
            Session Priority                   : 33

            Tracks not Committed to the R2 Side: 0
            Time that R2 is behind R1          : 00:00:00
            R1 Side Percent Cache In Use       :  0
            R2 Side Percent Cache In Use       :  0

            }
        }

root:/#
```

### Timfinder commands

#### Associate BCVs to a device group. There are two ways:

```
root:/# symbcv -sid xxxx -g dg_oradev_01 associate dev 0001
```

#### Establish the mirrors

```
root:/# symmir -g dg_oradev_01 -full establish DEV001 BCV001
```

#### Split operations.

```
root:/# symmir -g dg_oradev_01 split
```

There are several additional split modes and/or modifiers.

-   Instant

```
root:/# symmir -g dg_oradev_01 split -instant
```

-   Force

```
root:/# symmir -g dg_oradev_01 split -force
```

-   Differential

```
root:/# symmir -g dg_oradev_01 split -differential
```

-   Reverse

```
root/# symmir -g dg_oradev_01 reverse split
```

-   Reverse differential

```
root:/# symmir -g dg_oradev_01 reverse split -differential
```

#### Restore the BCV mirrors.

The restore operation will copy the data from the BCV to the Standard device.

-   Differential restore

```
root:/# symmir -g dg_oradev_01 restore
```

-   Full restore

```
root:/# symmir -g dg_oradev_01 -full restore
```

#### Reestablish operations.

It is very important to tell the difference between *Restore* and *Reestablish*. Reestablish will do a differential
update from the Standard device to the BCV device.

```
root:/# symmir -g dg_oradev_01 establish
```

#### Get the list of BCV devices

```
root:/# symbcv list

Symmetrix ID: 00029xxxxxxx
              BCV Device                   Standard Device          Status
------------------------------------ --------------------------- ------------
                              Inv.                        Inv.
Physical         Sym RDF Att. Tracks Physical        Sym  Tracks BCV <=> STD
------------------------------------ --------------------------- ------------
Not Visible      0030    (M)       0 N/A             N/A       0 NeverEstab
Not Visible      0031    (m)       - N/A             N/A       - NeverEstab
……………………………………………………………………………………………………………………………………………………………………………………………………………
c4t1d0s2         0088              0 c4t0d0s2        0084      0 Split
c4t1d1s2         0089              0 c4t0d1s2        0085      0 Split
c4t1d2s2         008A              0 c4t0d2s2        0086      0 Split
c4t1d3s2         008B              0 c4t0d3s2        0087      0 Split
```

#### Get the state of mirroring of the device pairs within a device group

```
root:/# /usr/symcli/bin/symmir -g dg_oracle_prod_01 query

Device Group (DG) Name: dg_oracle_prod_01
DG's Type             : RDF1
DG's Symmetrix ID     : 00029xxxxxxx

     Standard Device                    BCV Device                  State
-------------------------- ------------------------------------- ------------
                    Inv.                                  Inv.
Logical        Sym  Tracks Logical              Sym       Tracks STD <=> BCV
-------------------------- ------------------------------------- ------------

DEV001         0184      0 BCV001               039C *         0 Split
DEV002         0186      0 BCV002               039E *         0 Split
DEV003         0187      0 BCV003               039F *         0 Split
DEV004         0188      0 BCV004               03A0 *         0 Split
DEV005         0189      0 BCV005               03A1 *         0 Split
DEV006         018E      0 BCV006               03A6 *         0 Split
DEV007         018F      0 BCV007               03A7 *         0 Split
DEV008         0190      0 BCV008               03A8 *         0 Split
DEV009         0191      0 BCV009               03A9 *         0 Split
DEV010         01C7      0 BCV010               08A7 *         0 Split
DEV011         01CD      0 BCV011               08AA *         0 Split

Total              -------                               -------
  Track(s)               0                                     0
  MB(s)                0.0                                   0.0

Legend:

(*): The paired BCV device is associated with this group.

root:/#
```

#### List all BCV sessions in a Symmetrix array

```
root:/# symmir list -sid xxxx

Symmetrix ID: 00000000xxxx

  Standard Device       BCV Device              State
-------------------- ----------------------- --------------
           Invalid             Invalid   GBE
Sym        Tracks     Sym      Tracks         STD <=> BCV
-------------------- ----------------------- --------------
002B               0 0E0B            0   ... Synchronized
002E               0 0E00            0   ..X Synchronized
002E               0 0E0A            0   ... Synchronized
0032               0 0E0F            0   ... Split
00FF               0 00FD            0   ... Split
0DF5               0 0DA5            0   ..X Synchronized
0DF5               0 0DA4            0   ..X Synchronized
0F70               0 001B         3592   X.. SyncInProg
0F71               0 001C         4496   X.. SyncInProg
0F93               0 0DF9            0   ..X Split
1015               0 1069            0   X.. Synchronized

Total       --------          --------
  Tracks           0              8088
  MB(s)          0.0             505.5
```

And we are done. As I said this is not a full guide so please if there is anything that you don't get please leave a comment and I will try to clarify. Also if any of you have additional tips or recipes for Timefinder please comment.

Juanma.
