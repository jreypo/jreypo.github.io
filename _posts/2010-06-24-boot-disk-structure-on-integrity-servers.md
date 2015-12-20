---
layout: post
title: Boot disk structure on Integrity servers
date: 2010-06-24 00:25:37.000000000 +02:00
type: post
published: true
status: publish
categories:
- HP-UX
- Itanium
tags:
- EFI
- HP Service Partition
- HP-UX
- Itanium
meta:
  _edit_last: '11044808'
  geo_latitude: '40.416691'
  geo_longitude: '-3.700345'
  geo_accuracy: '17077'
  geo_address: Madrid
  geo_public: '1'
  _wp_old_slug: ''
  reddit: a:2:{s:5:"count";s:1:"0";s:4:"time";s:10:"1331008155";}
  _oembed_4a6e727b7a98c1bf0c1184012917206f: '{{unknown}}'
  _oembed_af944c719cea8a1ae26162f235b04aa8: '{{unknown}}'
author: juan_manuel_rey
---

The boot disk/disks of every Integrity server are divided into three partitions:

1.  EFI Partition: Contains the necessary tools and files to find and load the appropriate kernel. Here resides for example the `hpux.efi` utility.
2.  OS Partition: In the case of HP-UX contains the LVM or VxVM structure, the kernel and any filesystem that play a role during the boot process.
3.  HP Service Partition (HPSP).

### EFI Partition

The Extensible Firmware Interface (EFI) partition is subdivided into three main areas:

-   MBR: The Master Boot Record, located at the top of the disk, a legacy Intel structure ignored by EFI.
-   GPT: Every EFI partition is assigned a unique identifier known as GUID (Globally Unique Identifier). The locations of the GUID s are stored in the EFI GUID Partition Table or GPT. This very critical structure is replicated at the top and the bottom of the disk.
-   EFI System Partition: This partition contains the OS loader responsible of loading the operative system during the boot process. On HP-UX disks the OS loader is the famous`\efi\hpux\hpux.efi` file. Here is contained also `\efi\hpux\auto` file which stores the system boot string and some utilities as well.

### OS Partition

The OS Partition obviously contains the Operative System that runs on the server. An HP-UX partition contains a LIF area, private region and public region.

The Logical Interchange Format (LIF) boot area stores the following files:

-   ISL -  Not used on Integrity.
-   AUTO - Not used on Integrity.
-   HPUX -  Not used on Integrity.
-   LABEL  A binary file that contains the records of the locations of `/stand` and the primary swap.

The private region contains LVM and VxVM configuration information.

And the public region contains the corresponding volumes for:

-   stand: `/stand` filesystem including the HP-UX kernel.
-   swap: Primary swap space.
-   root: The root filesystem that includes `/`, `/etc`, `/dev` and `/sbin`.

### HP Service Partition

The HP Service Partition, or HPSP, is a FAT-32 filesystem that contains several offline diagnostic utilities to be used on unbootable systems.

Juanma.
