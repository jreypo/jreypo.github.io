---
layout: post
title: New HP-UX release, new HPVM, new features... same odd naming convention
date: 2010-04-06
type: post
published: true
status: publish
categories:
- HP-UX
tags:
- 11iv3
- HP-UX
- HPVM
- Integrity VMs
- LVM
- Opinion
author: juan_manuel_rey
comments: true
---

Last month HP released the last update of HP-UX 11iv3, the Update 6 or the March 2010 Update or 11.36... I decided some time ago to do not try to understand why we have a so stupid naming scheme for HP-UX.

Anyway, putting aside the philosophical rambling, HP-UX 11iv3 update 6 is here and it came full of [new features](http://bizsupport1.austin.hp.com/bc/docs/support/SupportManual/c02023869/c02023869.pdf). The following ones stand out, at least for me.

-   Improved system boot time thanks to RCEnhancement, [Oliver](http://omasse.blogspot.com/2010/03/hp-ux-11iv6-will-include-parallel-rc.html) wrote about it several weeks ago, until this release it was available in the [Software Depot](https://h20392.www2.hp.com/portal/swdepot/displayProductInfo.do?productNumber=RCEnh) and now is included in the install media.
-   New DRD version capable of synchronize the clone with the running system.
-   LVM updated to version 2.2. With this version we have logical volume snapshots, can't wait to try this :-D, logical volume migration within the same VG through the new *lvmove* command and boot support for LVM 2.2 volume groups, this is very cool because until this release `vg00` was stuck in LVM 1.0. Ignite-UX have also been updated to take advantage of this feature and we'll be asked to choose between LVM 1.0 and LVM 2.2 bootable volume groups.

This release comes bundled with the new HPVM version, the 4.2, which also adds a bunch of [new features](http://bizsupport2.austin.hp.com/bc/docs/support/SupportManual/c02023907/c02023907.pdf). To name a few.

-   Automatic memory reallocation.
-   Capacity of suspend and resume a guest.
-   Support for CVM/CFS backing stores for HPVM Serviceguard cluster packages.
-   Encryption of the data during Online VM migration.
-   AVIO support for Veritas Volume Manager based backing stores.

In short, there are a lot of new interesting features, a lot issues have also been fixed and as I said at the beginning we still have to live with an odd naming scheme but in the end I'm quite happy with this new release at least in theory because I didn't have the opportunity to try it yet but I will do very soon since I'm planning to deploy HPVM 4.2 in the near future. In fact my HPVM 3.5 to 4.1 migration project has become a 3.5 to 4.2 migration, how cool is that eh! ;-)

Juanma.
