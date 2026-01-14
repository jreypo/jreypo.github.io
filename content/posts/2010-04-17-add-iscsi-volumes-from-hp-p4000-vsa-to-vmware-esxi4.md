---
title: Add iSCSI volumes from HP P4000 VSA to VMware ESXi4
date: 2010-04-17
categories:
- Storage
- Sysadmin
- Virtualization
- VMware
tags:
- ESXi4
- HP Lefthand
- iSCSI
- P4000 VSA
- sysadmin
- systems administration
- VMware
- vSphere
showComments: true
---

In today's post I will try to explain step by step how to add an iSCSI volume from the HP Lefthand P4000 VSA to a VMware ESXi4 server.

## Step One: Create a volume

Lets suppose we already have a configured storage appliance, I showed how to create a cluster in my [previous post]({{< ref "posts/2010-04-09-first-hands-on-with-the-hp-lefthand-p4000-vsa.md" >}}) so I will not repeat that part here. Open the Centralized Management Console and go to **Management Group -> Cluster -> Volumes and Snapshots**.

[![](/images/volumes_snapshots.png)](/images/volumes_snapshots.png)

Click on **Tasks** and choose **New Volume**.

[![](/images/new_volume2.png)](/images/new_volume2.png)

Enter the volume name, a small description and the size. The volume can also be assigned to any server already connected to the cluster, as we don't have any server assigned this option can be ignored, for now.

[![](/images/volume_basic.png)](/images/volume_basic.png)

In the **Advanced** tab the volume can be assigned to an existing cluster and the RAID level, the volume type and the provisioning type can be set.

[![](/images/volume_advanced.png)](/images/volume_advanced.png)

When everything is configured click OK and the volume will be created. After the creation process, the new volume will be showed up in the CMC.

[![](/images/new_volume_created.png)](/images/new_volume_created.png)

At this point we have a new volume with some RAID protection level, none in this particular case since it's a single node cluster. Next step is to assign the volume to a server.

## Step Two: ESXi iSCSI configuration

Connect to the chosen ESXi4 Server through vSphere Client and from the **Configuration** tab in the right pane go to the **Networking** area and click the *Add Networking* link

[![](/images/esxi_add_networking.png)](/images/esxi_add_networking.png)

In the **Add Networking Wizard* select VMkernel as **Connection Type**.

[![](/images/wizard.png)](/images/wizard.png)

Create a new virtual switch.

[![](/images/new_vswicth.png)](/images/new_vswicth.png)

Enter the **Port Group Properties**, in my case the label as no other properties where relevant for me.

[![](/images/properties.png)](/images/properties.png)

Set the IP settings, go to **Summary** screen and click **Finish**.

[![IP Settings](/images/ip.png "IP Settings")](/images/ip.png)

The newly created virtual switch will appear in the **Networking** area.

[![](/images/vswitch_created.png)](/images/vswitch_created.png)

With the new virtual switch created go to **Storage Adapters** there you will see an iSCSI software adapter.

[![](/images/iscsi_soft_adapter.png)](/images/iscsi_soft_adapter.png)

Click on properties and on the **General** tab click the **Configure** button and check the **Enabled** status box.

[![](/images/enable_iscsi.png)](/images/enable_iscsi.png)

Once iSCSI is enabled its properties window will be populated.

[![](/images/iscsi_iqn.png)](/images/iscsi_iqn.png)

Click close, the server will ask for rescan of the adapter but at this point it is not necessary so it can be skipped.

## Step Three: Add the volume to the ESXi server

Now, that we have our volume created and the iSCSI adapter of our ESXi server activated, the next step is to add the storage to server.

On the HP Lefthand CMC go to the servers area add a new server.

[![](/images/new_server.png)](/images/new_server.png)

Add a name for the server, a small description, check the **Allow access via iSCSI** box and select the authentication. In the example I choose **CHAP not required**. With this option you only have to enter the **Initiator Node Name**, you can grab it from the details of the ESXi iSCSI adapter.

[![](/images/new_server_2.png)](/images/new_server_2.png)

To finish the process click OK and you will see the newly added server. Go to the **Volume and Snapshots** tab on the server configuration and from the **Tasks** menu assign a volume to the server.

[![](/images/assign_volume.png)](/images/assign_volume.png)

Select the volume created at the beginning,

[![](/images/assign_volume_2.png)](/images/assign_volume_2.png)

Now go back to the vSphere client and launch again the properties of the iSCSI adapter. On the **Dynamic Discovery** tab add the virtual IP address of the VSA cluster.

[![](/images/dynamic_discovery.png)](/images/dynamic_discovery.png)

Click **Close** and the server will ask again to re-scan the adapter, now say yes and after the re-scanning process the iSCSI LUN will show up.

[![](/images/iscsi_lun.png)](/images/iscsi_lun.png)

Now in the ESXi a new Datastore can be created with the newly configured storage. Of course the same LUN can also be used to provide shared storage for more ESXi servers and used for VMotion, HA or any other interesting VMware vSphere features. May be in another post ;-)

Juanma.
