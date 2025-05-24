---
title: Basic volume tasks with the HP Lefthand CLIQ command-line
date: 2010-06-16
type: post
classes: wide
published: true
status: publish
categories:
- Storage
tags:
- CLIQ
- HP Lefthand
- iSCSI
- P4000 VSA
author: juan_manuel_rey
comments: true
---

Following with the series of posts about the HP Lefthand SAN systems in this post I will explain the basic volume operations with CLIQ, the HP Lefthand SAN/iQ command-line.

I used the Lefthand VSA and the ESX4 servers from my home lab to illustrate the procedure. The commands are executed locally in the VSA via SSH. The following tasks will be covered:

- Volume creation.
- Assign a volume to one or more hosts.
- Volume deletion.

## Volume creation

The command to use is `createVolume`. The available options for this command are:

- `volumeName`
- `clusterName` - The cluster where the volume will be created on.
- `replication` -  The replication level from 1 (none) to 4 (4-way replication)
- `thinProvision` - 1 (Thin provisioning) or 2 (Full provision).
- `description`
- `size` - The size can be set in MB, GB or TB.

```
CLIQ>createVolume volumeName=vjm-cluster2 size=2GB clusterName=iSCSI-CL replication=1 thinProvision=1 description="vep01-02 datastore"

SAN/iQ Command Line Interface, v8.1.00.0047
(C) Copyright 2007-2009 Hewlett-Packard Development Company, L.P.

RESPONSE
 result         0
 processingTime 2539
 name           CliqSuccess
 description    Operation succeeded

CLIQ>
```

## Assign a volume to the hosts

The command to use in this task is *assignVolume*. Few parameters are accepted by this command:

- `volumeName`
- `ìnitiator` - The host/hosts IQNs.If the volume is going to be presented to more than one host the IQNs of the server must be separated by semicolons. One important tip, the operation must be done in one command, you can not assign the volume to a host in one command and to a new host in a second command, the last one will overwrite the first instead of adding the volume to one more host.
- Access wrights: The default is read-write (`rw`), read-only (`r`) or write-only (`w`) can also be set.

```
CLIQ>assignVolume volumeName=vjm-cluster2 initiator=iqn.1998-01.com.vmware:vep01-45602bf3;iqn.1998-01.com.vmware:vep02-5f779b32

SAN/iQ Command Line Interface, v8.1.00.0047
(C) Copyright 2007-2009 Hewlett-Packard Development Company, L.P.

RESPONSE
 result         0
 processingTime 4069
 name           CliqSuccess
 description    Operation succeeded

CLIQ>
```

And now that the volume is created and assigned to several servers check its configuration with `getVolumeInfo`.

```
CLIQ>getVolumeInfo volumeName=vjm-cluster2

SAN/iQ Command Line Interface, v8.1.00.0047
(C) Copyright 2007-2009 Hewlett-Packard Development Company, L.P.

RESPONSE
 result         0
 processingTime 1480
 name           CliqSuccess
 description    Operation succeeded

 VOLUME
 thinProvision  true
 stridePages    32
 status         online
 size           2147483648
 serialNumber   17a1c11e939940a4f7e91ee43654c94b000000000000006b
 scratchQuota   4194304
 reserveQuota   536870912
 replication    1
 name           vjm-cluster2
 minReplication 1
 maxSize        14587789312
 iscsiIqn       iqn.2003-10.com.lefthandnetworks:vdn:107:vjm-cluster2
 isPrimary      true
 initialQuota   536870912
 groupName      VDN
 friendlyName   
 description    vep01-02 datastore
 deleting       false
 clusterName    iSCSI-CL
 checkSum       false
 bytesWritten   18087936
 blockSize      1024
 autogrowPages  512

 PERMISSION
 targetSecret    
 loadBalance     true
 iqn             iqn.1998-01.com.vmware:vep01-45602bf3
 initiatorSecret
 chapRequired    false
 chapName        
 authGroup       vep01
 access          rw

 PERMISSION
 targetSecret    
 loadBalance     true
 iqn             iqn.1998-01.com.vmware:vep02-5f779b32
 initiatorSecret
 chapRequired    false
 chapName        
 authGroup       vep02
 access          rw

CLIQ>
```

If you refresh the storage configuration of the ESXs hosts through vSphere Client the new volume will be displayed.

[![](/assets/images/volume_on_esx.jpg "new volume")]({{site.url}}/assets/images/volume_on_esx.jpg)

## Volume deletion

Finally we are going to delete another volume that is no longer in use by the server of my lab.

```
CLIQ>deleteVolume volumeName=testvolume

SAN/iQ Command Line Interface, v8.1.00.0047
(C) Copyright 2007-2009 Hewlett-Packard Development Company, L.P.

This operation is potentially irreversible.  Are you sure? (y/n)

RESPONSE
 result         0
 processingTime 1416
 name           CliqSuccess
 description    Operation succeeded

CLIQ>
CLIQ>getvolumeinfo volumename=testvolume

SAN/iQ Command Line Interface, v8.1.00.0047
(C) Copyright 2007-2009 Hewlett-Packard Development Company, L.P.

RESPONSE
 result         8000100c
 processingTime 1201
 name           CliqVolumeNotFound
 description    Volume 'testvolume' not found

CLIQ>
```

And we are done. As always comments are welcome.

Juanma.
