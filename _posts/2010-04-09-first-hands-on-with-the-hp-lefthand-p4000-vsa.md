---
layout: post
title: First hands-on with the HP Lefthand P4000 VSA
date: 2010-04-09 00:41:25.000000000 +02:00
type: post
published: true
status: publish
categories:
- Storage
- Virtualization
tags:
- HP Lefthand
- iSCSI
- P4000 VSA
- Storage
- VMware Workstation
- vSphere
author: juan_manuel_rey
---

The **P400 Virtual Storage Appliance** is a storage product from HP, its features include:

-   Synchronous replication to remote sites.
-   Snapshot capability.
-   Thin provisioning.
-   Network RAID.

It can be used to create a virtual iSCSI SAN to provide shared storage for ESX environments.

A 60-day trial is available [here](http://www.hp.com/go/tryvsa), it requires to log-in with your HP Passport account. As I wanted to test it is what I did, there are two versions available one for ESX and a second one labeled as Laptop Demo which is optimized for VMware Workstation and also comes with the Centralized Management Console software. I choose the last one.

After the import the appliance into VMware Workstation you will see that it comes configured as *Other Linux 2.6x kernel* as guest OS, with 1 vCPU, 384MB of RAM, 2 308MB disks used for the OS of the VSA and a 7.2 GB disk. The three disks are SCSI and are configured as Independent-Persistent.

At that point I fired up the appliance and started to play with my VSA.

### First Step - Basic configuration.

A couple of minutes after have been started the appliance will show the log-in prompt.

[![VSA login prompt](/images/vsa-2010-04-05-01-26-15.png "VSA login prompt")]({{site.url}}/images/vsa-2010-04-05-01-26-15.png)

As instructed enter `start`. You will be redirected to another log-in screen where you only have to press *Enter* and then the configuration screen will appear.

[![VSA config screen](/images/vsa-2010-04-05-01-26-41.png "VSA config screen")]({{site.url}}/images/vsa-2010-04-05-01-26-41.png)

The first section `General Settings` allow you to create an administrator account and to set the password of the default account.

[![VSA Admin creation](/images/p4000-2-2010-04-08-23-21-46.png "VSA Admin creation")]({{site.url}}/images/p4000-2-2010-04-08-23-21-46.png)

Move to the networking settings. The first screen ask you to choose the network interface, in my case I only had one.

[![VSA network interface](/images/vsa-2010-04-05-01-29-07.png "VSA network interface")]({{site.url}}/images/vsa-2010-04-05-01-29-07.png)

And now you can proceed with the NIC configuration. Will ask for confirmation prior to commit any changes you made in the VSA network configuration.

[![VSA Network settings](/images/p4000-2-2010-04-09-00-28-32.png "VSA Network settings")]({{site.url}}/images/p4000-2-2010-04-09-00-28-32.png)

In the next area of the main menu, Network TCP Status, the speed of the network card can be forced to 1000MBs Full Duplex and the Frame Size can be set.

[![](/images/vsa-2010-04-05-01-32-15.png)]({{site.url}}/images/vsa-2010-04-05-01-32-15.png)

The final part is for group management configuration, in fact to remove the VSA from a management group, and to reset the VSA to its defaults settings.

Now we have our P4000 configured to be managed through CMC. I will not explain the CMC installation since it's just an almost *next -> next* task.

### Second step - VSA management with Centralized Management Console.

Launch the CMC. The **Find Nodes Wizard** will pop-up.

[![Find nodes wizard](/images/find_node_wizard.png "Find nodes wizard")]({{site.url}}/images/find_node_wizard.png)

Choose the appropriate option in the next screen. To add a single VSA choose the second one.

[![](/images/lefthand_02.png)]({{site.url}}/images/lefthand_02.png)

Enter the IP address of the appliance.

[![](/images/lefthand_08.png)]({{site.url}}/images/lefthand_08.png)

Click **Finish** and if the VSA is online the wizard will find it and add it to the CMC.

[![VSA Added to CMC](/images/lefthand_06.png "VSA Added to CMC")]({{site.url}}/images/lefthand_06.png)

Now the VSA is managed through the CMC but it is not part of a management group.

### Step Three - Add more storage.

The first basic tasks we're going to do with the VSA prior to Management Creation is to add more storage.

As the VSA is a virtual machine go to VMware Workstation or vSphere Client, depends on which VSA version are you using, and edit the appliance settings.

[![Add new disk](/images/p4000_add-new-disk_1.png)]({{site.url}}/images/p4000_add-new-disk_1.png)

If you look into the advanced configuration of the third disk, the 7.2GB one, you will see that it has the 1:0 SCSI address.

[![](/images/p4000_add-new-disk_2.png)]({{site.url}}/images/p4000_add-new-disk_2.png)

This is very important because the new disks must be added sequentially from 1:1 through 1:4 addresses in order to be detected by the VSA. If there is disk added to the VSA the 1:0 SCSI address must be used for the first disk.

Add the new disk and before finishing the process edit the advanced settings of the disk and set the SCSI address.

[![](/images/p4000_add-new-disk_8.png)]({{site.url}}/images/p4000_add-new-disk_8.png)

Now in the CMC go to the storage configuration. You will see the new disks as uninitialized.

[![](/images/p4000_add-new-disk_5.png)]({{site.url}}/images/p4000_add-new-disk_5.png)

Right click on the disk and select **Add Disk to RAID**.

[![](/images/p4000_add-new-disk_6.png)]({{site.url}}/images/p4000_add-new-disk_6.png)

Next you will see the disk active and already added to the RAID.

[![](/images/p4000_add-new-disk_7.png)]({{site.url}}/images/p4000_add-new-disk_7.png)

### Step four - Management group creation.

We're going to create the most basic configuration possible with a P4000 VSA. One VSA in a single management group and part of a single-node cluster.

From the **Getting Started** screen launch the **Management Groups, Clusters and Volume Wizard**.

[![Cluster Wizard](/images/cluster_wizard.png "Cluster Wizard")]({{site.url}}/images/cluster_wizard.png)

Select **New Management Group** and enter the data of the new group.

[![](/images/vsa_cluster4.png)]({{site.url}}/images/vsa_cluster4.png)

Add the administrative user.

[![](/images/vsa_cluster5.png)]({{site.url}}/images/vsa_cluster5.png)

Set the time of the Management Group.

[![](/images/vsa_cluster6.png)]({{site.url}}/images/vsa_cluster6.png)

Create a new Standard Cluster.

[![](/images/vsa_cluster7.png)]({{site.url}}/images/vsa_cluster7.png)

Enter the name of the cluster and select the nodes of the cluster, in this particular set-up there is only one node.

[![](/images/vsa_cluster8.png)]({{site.url}}/images/vsa_cluster8.png)

Finally add a virtual IP for the cluster.

[![](/images/vsa_cluster14.png)]({{site.url}}/images/vsa_cluster14.png)

Once the cluster is created the wizard will ask to create a volume. The volume can also be added later to the cluster.

[![](/images/vsa_cluster10.png)]({{site.url}}/images/vsa_cluster10.png)

After we click finish the management group and the cluster will be created.

[![](/images/vsa_cluster13.png "VSA_cluster13")]({{site.url}}/images/vsa_cluster13.png)

And we are done. In the next post about the P4000 I will show how to add and iSCSI volume to an ESXi4 server.

Juanma.
