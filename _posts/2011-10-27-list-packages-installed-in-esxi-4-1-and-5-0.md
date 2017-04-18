---
layout: post
title: List packages installed in ESXi 4.1 and 5.0
date: 2011-10-27
type: post
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- Busybox
- esxcli
- ESXi Shell
- ESXi TSM
- ESXi4
- ESXi5
- ipkg
- sysadmin
- systems administration
- Virtualization
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

Today a co-worker has asked me how to list the packages installed in an ESXi 4.1 Update 1 server, in the ESX COS we had the RedHat `rpm` command but in ESXi there is no `rpm` and of course there is no COS.

His intention was to look for the version of the `qla2xxx` driver and my first thought was to use `vmkload_mod`, the problem is that with this command you can get the version of a driver already loaded by the VMkernel and we wanted to look for the version of a driver installed but no loaded.

I tried `esxupdate` with no luck.

```
~ # esxupdate query
----Bulletin ID----- -----Installed----- --------------Summary---------------
ESXi410-201101223-UG 2011-01-13T05:09:39 3w-9xxx: scsi driver for VMware ESXi
ESXi410-201101224-UG 2011-01-13T05:09:39 vxge: net driver for VMware ESXi    
~ #
```

Then I suddenly thought that the ESXi Tech Support Mode is based on Busybox. If you have ever use a Busybox environment, like a QNAP NAS, you will probably remember that the way to install new software over the network is with `ipkg` command and to list the software packages already installed the syntax is `ipkg list_installed`.

```
~ # ipkg list_installed
emulex-cim-provider - 410.2.0.32.1-207424 -
lsi-provider - 410.04.V0.24-140815 -
qlogic-fchba-provider - 400.1.1.8-140815 -
vmware-esx-drivers-ata-libata - 400.2.00.1-1vmw.1.4.348481 -
vmware-esx-drivers-ata-pata-amd - 400.0.2.4.1-1vmw.1.4.348481 -
vmware-esx-drivers-ata-pata-atiixp - 400.0.4.3.1-1vmw.1.4.348481 -
vmware-esx-drivers-ata-pata-cmd64x - 400.0.2.1.1-1vmw.1.4.348481 -
vmware-esx-drivers-ata-pata-hpt3x2n - 400.0.3.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-ata-pata-pdc2027x - 400.0.74ac5.1-1vmw.1.4.348481 -
vmware-esx-drivers-ata-pata-serverworks - 400.0.3.7.1-1vmw.1.4.348481 -
vmware-esx-drivers-ata-pata-sil680 - 400.0.3.2.1-1vmw.1.4.348481 -
vmware-esx-drivers-ata-pata-via - 400.0.1.14.1-1vmw.1.4.348481 -
vmware-esx-drivers-block-cciss - 400.3.6.14.10.1-2vmw.1.4.348481 -
vmware-esx-drivers-char-hpcru - 400.1.1.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-char-pseudo-char-dev - 400.0.0.1.1-1vmw.1.4.348481 -
vmware-esx-drivers-char-random - 400.1.0.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-char-tpm-tis - 400.0.0.1.1-1vmw.1.4.348481 -
vmware-esx-drivers-ehci-ehci-hcd - 400.1.0.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-hid-hid - 400.2.6.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-ioat-ioat - 400.2.15.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-ipmi-ipmi-devintf - 400.39.2.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-ipmi-ipmi-msghandler - 400.39.2.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-ipmi-ipmi-si-drv - 400.39.2.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-net-bnx2 - 400.2.0.7d-3vmw.1.4.348481 -
vmware-esx-drivers-net-bnx2x - 400.1.54.1.v41.1-2vmw.1.4.348481 -
vmware-esx-drivers-net-cdc-ether - 400.1.0.0.1-2vmw.1.4.348481 -
vmware-esx-drivers-net-cnic - 400.1.9.7d.rc2.3.1-2vmw.1.4.348481 -
vmware-esx-drivers-net-e1000 - 400.8.0.3.2-1vmw.1.4.348481 -
vmware-esx-drivers-net-e1000e - 400.1.1.2.1-1vmw.1.4.348481 -
vmware-esx-drivers-net-enic - 400.1.4.0.261-1vmw.1.4.348481 -
vmware-esx-drivers-net-forcedeth - 400.0.61.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-net-igb - 400.1.3.19.12.2-2vmw.1.4.348481 -
vmware-esx-drivers-net-ixgbe - 400.2.0.38.2.5.1-1vmw.1.4.348481 -
vmware-esx-drivers-net-nx-nic - 400.4.0.550.1-1vmw.1.4.348481 -
vmware-esx-drivers-net-s2io - 400.2.1.4.13427.1-1vmw.1.4.348481 -
vmware-esx-drivers-net-sky2 - 400.1.20-1vmw.1.4.348481 -
vmware-esx-drivers-net-tg3 - 400.3.86.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-net-usbnet - 400.1.0.0.1-2vmw.1.4.348481 -
vmware-esx-drivers-ohci-usb-ohci - 400.1.0.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-sata-ahci - 400.2.0.0.1-5vmw.1.4.348481 -
vmware-esx-drivers-sata-ata-piix - 400.2.00ac6.1-3vmw.1.4.348481 -
vmware-esx-drivers-sata-sata-nv - 400.2.0.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-sata-sata-promise - 400.1.04.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-sata-sata-sil - 400.2.0.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-sata-sata-svw - 400.2.0.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-scsi-aacraid - 400.4.1.1.5.1-1vmw.1.4.348481 -
vmware-esx-drivers-scsi-adp94xx - 400.1.0.8.12.1-1vmw.1.4.348481 -
vmware-esx-drivers-scsi-aic79xx - 400.3.2.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-scsi-bnx2i - 400.1.8.11t5.rc2.8.1-4vmw.1.4.348481 -
vmware-esx-drivers-scsi-fnic - 400.1.1.0.113.2-4vmw.1.4.348481 -
vmware-esx-drivers-scsi-hpsa - 400.3.6.14.45-4vmw.1.4.348481 -
vmware-esx-drivers-scsi-ips - 400.7.12.06.1-3vmw.1.4.348481 -
vmware-esx-drivers-scsi-iscsi-linux - 400.1.0.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-scsi-lpfc820 - 400.8.2.1.30.1-58vmw.1.4.348481 -
vmware-esx-drivers-scsi-megaraid-mbox - 400.2.20.5.1.4-1vmw.1.4.348481 -
vmware-esx-drivers-scsi-megaraid-sas - 400.4.0.14.1-18vmw.1.4.348481 -
vmware-esx-drivers-scsi-megaraid2 - 400.2.00.4.1-4vmw.1.4.348481 -
vmware-esx-drivers-scsi-mpt2sas - 400.04.255.03.00.1-6vmw.1.4.348481 -
vmware-esx-drivers-scsi-mptsas - 400.4.21.00.01.1-6vmw.1.4.348481 -
vmware-esx-drivers-scsi-mptspi - 400.4.21.00.01.1-6vmw.1.4.348481 -
vmware-esx-drivers-scsi-qla2xxx - 400.831.k1.28.1-1vmw.1.4.348481 -
vmware-esx-drivers-scsi-qla4xxx - 400.5.01.03.1-10vmw.1.4.348481 -
vmware-esx-drivers-scsi-sample-iscsi - 400.1.0.0-1vmw.1.4.348481 -
vmware-esx-drivers-uhci-usb-uhci - 400.3.0.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-usb-storage-usb-storage - 400.1.0.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-usbcore-usb - 400.1.0.0.1-1vmw.1.4.348481 -
vmware-esx-drivers-vmklinux-vmklinux - 4.1.0-1.4.348481 -
Successfully terminated.
~ #
```

There you are! There is one gotcha to get the version, it starts just after the 400.

Next task of course was to do the same in ESXi 5.0.

```
~ # ipkg list_installed
-sh: ipkg: not found
~ #
```

Ouch! `ipkg` has been removed from ESXi 5.0. The key to get the same list is `esxcli`.

```
~ # esxcli software vib list
Name                  Version                             Vendor  Acceptance Level  Install Date
--------------------  ----------------------------------  ------  ----------------  ------------
ata-pata-amd          0.3.10-3vmw.500.0.0.469512          VMware  VMwareCertified   2011-09-07 
ata-pata-atiixp       0.4.6-3vmw.500.0.0.469512           VMware  VMwareCertified   2011-09-07 
ata-pata-cmd64x       0.2.5-3vmw.500.0.0.469512           VMware  VMwareCertified   2011-09-07 
ata-pata-hpt3x2n      0.3.4-3vmw.500.0.0.469512           VMware  VMwareCertified   2011-09-07 
ata-pata-pdc2027x     1.0-3vmw.500.0.0.469512             VMware  VMwareCertified   2011-09-07 
ata-pata-serverworks  0.4.3-3vmw.500.0.0.469512           VMware  VMwareCertified   2011-09-07 
ata-pata-sil680       0.4.8-3vmw.500.0.0.469512           VMware  VMwareCertified   2011-09-07 
ata-pata-via          0.3.3-2vmw.500.0.0.469512           VMware  VMwareCertified   2011-09-07 
block-cciss           3.6.14-10vmw.500.0.0.469512         VMware  VMwareCertified   2011-09-07 
ehci-ehci-hcd         1.0-3vmw.500.0.0.469512             VMware  VMwareCertified   2011-09-07 
esx-base              5.0.0-0.0.469512                    VMware  VMwareCertified   2011-09-07 
esx-tboot             5.0.0-0.0.469512                    VMware  VMwareCertified   2011-09-07 
ima-qla4xxx           2.01.07-1vmw.500.0.0.469512         VMware  VMwareCertified   2011-09-07 
ipmi-ipmi-devintf     39.1-4vmw.500.0.0.469512            VMware  VMwareCertified   2011-09-07 
ipmi-ipmi-msghandler  39.1-4vmw.500.0.0.469512            VMware  VMwareCertified   2011-09-07 
ipmi-ipmi-si-drv      39.1-4vmw.500.0.0.469512            VMware  VMwareCertified   2011-09-07 
misc-cnic-register    1.1-1vmw.500.0.0.469512             VMware  VMwareCertified   2011-09-07 
misc-drivers          5.0.0-0.0.469512                    VMware  VMwareCertified   2011-09-07 
net-be2net            4.0.88.0-1vmw.500.0.0.469512        VMware  VMwareCertified   2011-09-07 
net-bnx2              2.0.15g.v50.11-5vmw.500.0.0.469512  VMware  VMwareCertified   2011-09-07 
net-bnx2x             1.61.15.v50.1-1vmw.500.0.0.469512   VMware  VMwareCertified   2011-09-07 
net-cnic              1.10.2j.v50.7-2vmw.500.0.0.469512   VMware  VMwareCertified   2011-09-07 
net-e1000             8.0.3.1-2vmw.500.0.0.469512         VMware  VMwareCertified   2011-09-07 
net-e1000e            1.1.2-3vmw.500.0.0.469512           VMware  VMwareCertified   2011-09-07 
net-enic              1.4.2.15a-1vmw.500.0.0.469512       VMware  VMwareCertified   2011-09-07 
net-forcedeth         0.61-2vmw.500.0.0.469512            VMware  VMwareCertified   2011-09-07 
net-igb               2.1.11.1-3vmw.500.0.0.469512        VMware  VMwareCertified   2011-09-07 
net-ixgbe             2.0.84.8.2-10vmw.500.0.0.469512     VMware  VMwareCertified   2011-09-07 
net-nx-nic            4.0.557-3vmw.500.0.0.469512         VMware  VMwareCertified   2011-09-07 
net-r8168             8.013.00-3vmw.500.0.0.469512        VMware  VMwareCertified   2011-09-07 
net-r8169             6.011.00-2vmw.500.0.0.469512        VMware  VMwareCertified   2011-09-07 
net-s2io              2.1.4.13427-3vmw.500.0.0.469512     VMware  VMwareCertified   2011-09-07 
net-sky2              1.20-2vmw.500.0.0.469512            VMware  VMwareCertified   2011-09-07 
net-tg3               3.110h.v50.4-4vmw.500.0.0.469512    VMware  VMwareCertified   2011-09-07 
ohci-usb-ohci         1.0-3vmw.500.0.0.469512             VMware  VMwareCertified   2011-09-07 
sata-ahci             3.0-6vmw.500.0.0.469512             VMware  VMwareCertified   2011-09-07 
sata-ata-piix         2.12-4vmw.500.0.0.469512            VMware  VMwareCertified   2011-09-07 
sata-sata-nv          3.5-3vmw.500.0.0.469512             VMware  VMwareCertified   2011-09-07 
sata-sata-promise     2.12-3vmw.500.0.0.469512            VMware  VMwareCertified   2011-09-07 
sata-sata-sil         2.3-3vmw.500.0.0.469512             VMware  VMwareCertified   2011-09-07 
sata-sata-svw         2.3-3vmw.500.0.0.469512             VMware  VMwareCertified   2011-09-07 
scsi-aacraid          1.1.5.1-9vmw.500.0.0.469512         VMware  VMwareCertified   2011-09-07 
scsi-adp94xx          1.0.8.12-6vmw.500.0.0.469512        VMware  VMwareCertified   2011-09-07 
scsi-aic79xx          3.1-5vmw.500.0.0.469512             VMware  VMwareCertified   2011-09-07 
scsi-bnx2i            1.9.1d.v50.1-3vmw.500.0.0.469512    VMware  VMwareCertified   2011-09-07 
scsi-fnic             1.5.0.3-1vmw.500.0.0.469512         VMware  VMwareCertified   2011-09-07 
scsi-hpsa             5.0.0-17vmw.500.0.0.469512          VMware  VMwareCertified   2011-09-07 
scsi-ips              7.12.05-4vmw.500.0.0.469512         VMware  VMwareCertified   2011-09-07 
scsi-lpfc820          8.2.2.1-18vmw.500.0.0.469512        VMware  VMwareCertified   2011-09-07 
scsi-megaraid-mbox    2.20.5.1-6vmw.500.0.0.469512        VMware  VMwareCertified   2011-09-07 
scsi-megaraid-sas     4.32-1vmw.500.0.0.469512            VMware  VMwareCertified   2011-09-07 
scsi-megaraid2        2.00.4-9vmw.500.0.0.469512          VMware  VMwareCertified   2011-09-07 
scsi-mpt2sas          06.00.00.00-5vmw.500.0.0.469512     VMware  VMwareCertified   2011-09-07 
scsi-mptsas           4.23.01.00-5vmw.500.0.0.469512      VMware  VMwareCertified   2011-09-07 
scsi-mptspi           4.23.01.00-5vmw.500.0.0.469512      VMware  VMwareCertified   2011-09-07 
scsi-qla2xxx          901.k1.1-14vmw.500.0.0.469512       VMware  VMwareCertified   2011-09-07 
scsi-qla4xxx          5.01.03.2-3vmw.500.0.0.469512       VMware  VMwareCertified   2011-09-07 
uhci-usb-uhci         1.0-3vmw.500.0.0.469512             VMware  VMwareCertified   2011-09-07 
tools-light           5.0.0-0.0.469512                    VMware  VMwareCertified   2011-09-07 
~ #
```

A final thought for all of you starting with vSphere 5, `esxcli` is the key in ESXi 5.0 shell.

Juanma.
