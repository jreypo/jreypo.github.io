---
title: iSCSI initiator configuration in RedHat Enterprise Linux 5
date: 2011-02-22
tags:
- linux
- storage
- sysadmin
showComments: true
---

The following post will discuss about **iSCSI initiator** configuration in **RedHat Enterprise Linux 5**, this method is also applicable to all RHEL5 derivatives. The iSCSI LUNs will be provided by an HP P4000 array.

First of all we need to get and install the `iscsi-initiator-utils` RPM package, you can use yum to get and install the package from any supported repository for [CentOS](http://www.centos.org/) or RHEL. You can also download the package from [RedHat Network](http://rhn.redhat.com/) if you have a valid RHN account and your system doesn't have internet connection.

```text
[root@rhel5 ~]# rpm -ivh /tmp/iscsi-initiator-utils-6.2.0.871-0.16.el5.x86_64.rpm
Preparing...                ########################################### [100%]
   1:iscsi-initiator-utils  ########################################### [100%]
[root@rhel5 ~]#
[root@rhel5 ~]#rpm -qa | grep iscsi
iscsi-initiator-utils-6.2.0.871-0.16.el5
[root@rhel5 ~]# rpm -qi iscsi-initiator-utils-6.2.0.871-0.16.el5
Name        : iscsi-initiator-utils        Relocations: (not relocatable)
Version     : 6.2.0.871                         Vendor: Red Hat, Inc.
Release     : 0.16.el5                      Build Date: Tue 09 Mar 2010 09:16:29 PM CET
Install Date: Wed 16 Feb 2011 11:34:03 AM CET      Build Host: x86-005.build.bos.redhat.com
Group       : System Environment/Daemons    Source RPM: iscsi-initiator-utils-6.2.0.871-0.16.el5.src.rpm
Size        : 1960412                          License: GPL
Signature   : DSA/SHA1, Wed 10 Mar 2010 04:26:37 PM CET, Key ID 5326810137017186
Packager    : Red Hat, Inc. <http://bugzilla.redhat.com/bugzilla>
URL         : http://www.open-iscsi.org
Summary     : iSCSI daemon and utility programs
Description :
The iscsi package provides the server daemon for the iSCSI protocol,
as well as the utility programs used to manage it. iSCSI is a protocol
for distributed disk access using SCSI commands sent over Internet
Protocol networks.
[root@rhel5 ~]#
```

Next we are going to configure the initiator. The iSCSI initiator is composed by two services, `iscsi` and `iscsid`, enable them to start at system startup using `chkconfig`.

```text
[root@rhel5 ~]# chkconfig iscsi on
[root@rhel5 ~]# chkconfig iscsid on
[root@rhel5 ~]#
[root@rhel5 ~]# chkconfig --list | grep iscsi
iscsi           0:off   1:off   2:on    3:on    4:on    5:on    6:off
iscsid          0:off   1:off   2:on    3:on    4:on    5:on    6:off
[root@rhel5 ~]#
```

Once iSCSI is configured start the service.

```text
[root@rhel5 ~]# service iscsi start
iscsid is stopped
Starting iSCSI daemon:                                     [  OK  ]
                                                           [  OK  ]
Setting up iSCSI targets: iscsiadm: No records found!
                                                           [  OK  ]
[root@rhel5 ~]#
[root@rhel5 ~]# service iscsi status
iscsid (pid  14170) is running...
[root@rhel5 ~]#
```

From P4000 CMC we need to add the server to the management group configuration like we would do with any other server.

[![](/images/p4000-addserver.png "P4000 CMC add server")](/images/p4000-addserver.png)

The server iqn can be found in the file `/etc/iscsi/initiatorname.iscsi`.

```text
[root@cl-node1 ~]# cat /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.1994-05.com.redhat:2551bf29b48
[root@cl-node1 ~]#
```

Create any iSCSI volumes you need in the P4000 arrays and assign them to the RedHat system. Then to discover the presented LUNs, from the Linux server run the `iscsiadm` command.

```text
[root@rhel5 ~]# iscsiadm -m discovery -t sendtargets -p 192.168.126.60
192.168.126.60:3260,1 iqn.2003-10.com.lefthandnetworks:mlab:62:lv-rhel01
[root@rhel5 ~]#
```

Restart the iSCSI initiator to make the new block device available to the operative system.

```text
[root@rhel5 ~]# service iscsi restart
Stopping iSCSI daemon:
iscsid dead but pid file exists                            [  OK  ]
Starting iSCSI daemon:                                     [  OK  ]
                                                           [  OK  ]
Setting up iSCSI targets: Logging in to [iface: default, target: iqn.2003-10.com.lefthandnetworks:mlab:62:lv-rhel01, portal: 192.168.126.60,3260]
Login to [iface: default, target: iqn.2003-10.com.lefthandnetworks:mlab:62:lv-rhel01, portal: 192.168.126.60,3260]: successful
                                                           [  OK  ]
[root@rhel5 ~]#
```

Then check that the new disk is available, I used `lsscsi` but `fdisk -l` will do the trick too.

```text
[root@rhel5 ~]# lsscsi
[0:0:0:0]    disk    VMware,  VMware Virtual S 1.0   /dev/sda
[2:0:0:0]    disk    LEFTHAND iSCSIDisk        9000  /dev/sdb
[root@rhel5 ~]#
[root@rhel5 ~]# fdisk -l /dev/sdb

Disk /dev/sdb: 156.7 GB, 156766306304 bytes
255 heads, 63 sectors/track, 19059 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

Disk /dev/sdb doesn't contain a valid partition table
[root@rhel5 ~]#
```

At this point the iSCSI configuration is done, the new LUNs will be available through a system reboot as long as the iSCSI service is enabled.

Juanma.
