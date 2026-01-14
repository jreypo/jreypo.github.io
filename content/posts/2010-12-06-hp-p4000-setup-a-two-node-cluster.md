---
title: 'HP P4000: Setup a two-node cluster'
date: 2010-12-06
tags:
- storage
- vmware
- vsphere
showComments: true
---

This post will outline the necessary steps to create a standard (no-multisite) **HP P4000** cluster with two nodes. Creating a two-node cluster is a very similar process as the one-node cluster described in my first [post]({{< ref "posts/2010-04-09-first-hands-on-with-the-hp-lefthand-p4000-vsa.md" >}}) about P4000 systems.

The cluster is composed by:

- 2 HP P4000 Virtual Storage Appliances
- 1 HP P4000 Failover Manager

The **Failover Manager**, or FOM, is a specialized version of the SAN/iQ software. It runs as a virtual appliance in VMware, thought the most common situation is to run it in a ESX/ESXi servers running it under VMware player or Server is also supported.

The FOM integrates into a management group as a real manager and is intended only to provide quorum to the cluster, one of its main purposes is to provide quorum in Multi-site clusters. I decided to use it in this post to provide an example as real as possible.

To setup this cluster I used virtual machines inside [VMware Workstation](http://www.vmware.com/products/workstation/index.html), but the same design can also be created with physical servers and P4000 storage systems.

From the **Getting started** screen launch the clusters wizard.

[![](/images/vjm-p4000_thumb.png "P4000 Getting Started")](/images/vjm-p4000.png)

Select the two P4000 storage systems and enter the name of the Management Group

[![](/images/new_mgmt-group.png "New Management Group")](/images/new_mgmt-group.png)

During the group creation will ask to create a cluster, choose the two nodes as members of the cluster, will add the FOM later, and assign a name to the cluster.

[![](/images/clusterlab01.png "Cluster")](/images/clusterlab01.png)

Next assign a virtual IP address to the cluster.

[![](/images/clusterlab01_02.png "Cluster VIP address")](/images/clusterlab01_02.png)

Enter the administrative level credentials for the cluster.

[![](/images/admin_user.png "Admin user creation")](/images/admin_user.png)

Finally the wizard will ask if you want to create volumes in the cluster, I didn't take that option and finished the cluster creation process. You can also add the volumes later as I described in one of my previous [posts]({{< ref "posts/2010-04-17-add-iscsi-volumes-from-hp-p4000-vsa-to-vmware-esxi4.md" >}} "Add iSCSI volumes from HP P4000 VSA to VMware ESXi4").

Now the that cluster is formed we are going to add the Failover Manager.

It's is important that the FOM requires the same configuration as any VSA as I depicted in my first post about the P4000 storage systems.

In the Central Management Console right-click into the FOM and select **Add to existing management group**.

[![](/images/fom_join.png "FOM join group")](/images/fom_join.png)

Select the management group and click **Add**.

[![](/images/fom_join2.png "Select management group")](/images/fom_join2.png)

With this operation the cluster configuration is done. If everything went well in the end you should have something like this.

[![](/images/fom_join3.png)](/images/fom_join3.png)

Juanma.
