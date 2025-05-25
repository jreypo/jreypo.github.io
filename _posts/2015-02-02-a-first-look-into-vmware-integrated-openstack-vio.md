---
title: 'A first look into VMware Integrated #OpenStack (VIO)'
date: 2015-02-02
type: post
classes: wide
published: true
status: publish
categories:
- OpenStack
- VMware
tags:
- Cloud
- DVS
- network virtualization
- NSX
- NSX for vSphere
- OpenStack
- SDDC
- VIO
- VMware
- VMware Integrated OpenStack
- VSAN
- vSphere
author: juan_manuel_rey
comments: true
image:
  feature: openstack-banner.jpg
---

**[VMware Integrated OpenStack](http://www.vmware.com/products/openstack),** or VIO, was announced during last year VMworld in San Francisco and has been finally released today by VMware.

For me this is a very special release because I have been one of the lucky internal adopters and beta testers of VIO. I have spent many hours working with several VIO builds and trying to help our incredible engineering team. This is in my opinion a really solid and well designed product and will be a game changer in the OpenStack world. Honestly I
cannot be more excited :)

[![](/assets/images/image01.png)]({{site.url}}/assets/images/image01.png)

VIO is basically a VMware supported OpenStack distribution prepared to run on top of an existing VMware infrastructure. VMware Integrated OpenStack will empower any VMware Administrator to easily deliver and operate an Enterprise production grade OpenStack cloud on VMware components. This means that you will be able at to take advantage of all VMware vSphere great features like HA, DRS or VSAN for your OpenStack cloud and also extend and integrate it with other VMware management components like [vRealize Operations](http://www.vmware.com/ap/products/vrealize-operations) and [vRealize Log Insight](http://www.vmware.com/products/vrealize-log-insight).

## VIO components

VIO is made by two main building blocks, first the VIO Manager and second OpenStack components. VIO is packaged as an OVA file that contains the VIO Manager server and an Ubuntu Linux virtual machine to be used as the template for the different OpenStack components.

The OpenStack services in VIO are deployed as a distributed highly available solution formed by the following components:

- OpenStack controllers. Two virtual machines running Horizon Dashboard, Nova (API, scheduler and VNC) services, Keystone, Heat, Glance, and Cinder services in an active-active cluster.
- Memcached cluster.
- RabbitMQ cluster, for messaging services used by all OpenStack services.
- Load Balancer virtual machines, an active-active cluster managing the internal and public virtual IP addresses.
- Nova Compute machine, running the n-cpu service.
- Database cluster. A three node MariaDB Galera cluster that stores the OpenStack metadata.
- Object Storage machine, running Swift services.
- DHCP nodes. These nodes are only required if NSX is not selected as provider for Neutron.

## Installation requirements

To be able to successfully deploy VIO you will need at least the following:

- One management cluster with two to three hosts, depending on the hardware resources of the hosts.
- One Edge cluster. As with any NSX for vSphere deployment it is recommended to deploy a separate cluster to run all Edge gateway instances.
- One compute cluster to be used by Nova to run instances. One ESXi host will be enough but again that will depend on how much resources are available and what kind of workloads you want to run.
- Management network with at least 15 static IP addresses available.
- External network with a minimum of two IP addresses available. This is the network where Horizon portal will be exposed and that will be used by the tenants to access OpenStack APIs and services.
- Data network, only needed if NSX is going to be used. The different tenant logical network will be created on top of this, the management network can be used but it is recommended to have a     separate network.
- NSX for vSphere, 6.1.2 at minimum. It has to be setup prior to VIO deployment if NSX plugin is going to be used with Neutron.
- Distributed Port Group. In case of choosing DVS-based networking a vSphere port-group tagged with VLAN 4095 must be setup. This port group will be used as the data network.

The hardware requirements are around 56 vCPU, 192GB of memory and 605GB of storage. To that you have to add NSX for vSphere required resources for the NSX Manager, the three NSX Controllers and the NSX Edge pool, if NSX is going to be used.

Anyway in a future post I will review in detail all the pre-requisites and their setup process for VIO, and the integration between NSX-v and Neutron.

## VIO Installation

Now that we have seen a bit of VIO I am going to show how to perform an installation.

### Deploying VIO Manager

The first step is to deploy VIO OVA on our management cluster. From vSphere Web Client launch the **Deploy OVF Template** wizard and enter the URL to the VIO OVA file.

[![](/assets/images/image02.png)]({{site.url}}/assets/images/image02.png)

Accept the EULA and proceed to configure the template. First as with any OVA template enter the name and the folder,

[![](/assets/images/image03.png)]({{site.url}}/assets/images/image03.png)

Select the datastore and the storage format.

[![](/assets/images/image04.png)]({{site.url}}/assets/images/image04.png)

Select the network for VIO Manager.

[![](/assets/images/image05.png)]({{site.url}}/assets/images/image05.png)

Now we will customize the template, this includes entering the VIO Manager server networking settings, NTP, SSO lookup service URL and Syslog server.

[![](/assets/images/screen-shot-2015-01-31-at-00-32-27.png)]({{site.url}}/assets/images/screen-shot-2015-01-31-at-00-32-27.png)

Go through the next two screens, click finish and start the deployment. Once it is finished you will have a new vApp with the two virtual machines. Our next step is to register the management server with vCenter, power on the OMS vApp and when the management server is fully started logout of vSphere Web Client. Log in back to vSphere Web Client, you will notice a new icon in the Home page.

[![](/assets/images/screen-shot-2015-02-01-at-02-34-35.png)]({{site.url}}/assets/images/screen-shot-2015-02-01-at-02-34-35.png)

Access the VIO plugin interface and in the Summary you should see that VIO Manager has automatically registered itself with vCenter.

[![](/assets/images/image08.png)]({{site.url}}/assets/images/image08.png)

From this screen you can also change the VIO Manager server in case you need to re-deploy a new one. To do so select the management server in the pop-up and click **OK**.

[![](/assets/images/screen-shot-2015-01-31-at-17-24-49.png)]({{site.url}}/assets/images/screen-shot-2015-01-31-at-17-24-49.png)

Accept the SSL certificate to finish the procedure.

[![](/assets/images/screen-shot-2014-08-23-at-01-41-07.png)]({{site.url}}/assets/images/screen-shot-2014-08-23-at-01-41-07.png)

VIO Manager Server will now be displayed as connected in the **Summary** tab.

[![](/assets/images/image11.png)]({{site.url}}/assets/images/image11.png)

### Deploying OpenStack

With VIO Manager running and connected to our vCenter it is time now to deploy OpenStack. Proceed to the *Getting Started* tab and click **Deploy OpenStack**.

[![](/assets/images/image12.png)]({{site.url}}/assets/images/image12.png)

A new wizard will be launched. In the first screen we must select the deployment type. VIO allows to deploy a new OpenStack installation or deploy from a previously saved template file.

[![](/assets/images/image13.png)]({{site.url}}/assets/images/image13.png)

Provide the vCenter administrative credentials.

[![](/assets/images/image14.png)]({{site.url}}/assets/images/image14.png)

Select the management cluster where we are going to deploy VIO.

[![](/assets/images/image15.png)]({{site.url}}/assets/images/image15.png)

Next you need to configure the Management and External networks. Select the appropriate vSphere port-groups for each network and fill in the network ranges, gateway, netmask and DNS server fields.

[![](/assets/images/image16.png)]({{site.url}}/assets/images/image16.png)

Enter the values for the load balancer configuration:

- Public Virtual IP address
- Public Hostname, this hostname must resolve to the Public IP address.

[![](/assets/images/image17.png)]({{site.url}}/assets/images/image17.png)

Add a cluster to be used for Nova.

[![](/assets/images/image18.png)]({{site.url}}/assets/images/image18.png)

Add the datastores to be used by Nova to store the different instances. If you have a [VSAN](http://www.vmware.com/products/virtual-san) datastore keep in mind that to be able to use it with Nova the images stored in Glance have to be [streamOptimzed](https://blueprints.launchpad.net/nova/+spec/vmware-vsan-support).

[![](/assets/images/image19.png)]({{site.url}}/assets/images/image19.png)

Select the datastore to be used by Glance image service.

[![](/assets/images/image20.png)]({{site.url}}/assets/images/image20.png)

Configure Neutron networking. For Neutron there are two different options:

- DVS-based networking
- NSX networking

For DVS simply select the Virtual Distributed Switch where you created the port-group for the data network with the VLAN 4095 configured.

For NSX deployment you must enter:

- NSX Manager IP address.
- NSX Manager administrative username.
- NSX Manager administrative user password.
- VDN Scope. Basically the Transport Zone in NSX-v to be used as transport layer for data traffic.
- Edge Cluster. A vSphere cluster to deploy the NSX Edge instances.
- Virtual Distributed Switch for NSX networking.
- External Network. This a port group to be used as external network by instances in OpenStack via a virtual router. This port group should be accessible from compute, management and edge clusters.

[![](/assets/images/image21.png)]({{site.url}}/assets/images/image21.png)

During the Neutron configuration the wizard will connect to the NSX Manager with the provided credentials and will ask to accept the SSL certificate.

[![](/assets/images/image22.png)]({{site.url}}/assets/images/image22.png)

In the next screen the wizard will ask for the OpenStack admin user, password and project. Also you can select the Keystone type option:

- Database
- Active Directory as LDAP Server.

[![](/assets/images/image23.png)]({{site.url}}/assets/images/image23.png)

Finally set the syslog server, it is not mandatory to set this value but it is highly recommended.

[![](/assets/images/image24.png)]({{site.url}}/assets/images/image24.png)

Review the configuration and click **Finish**.

[![](/assets/images/screen-shot-2015-01-31-at-20-43-54.png)]({{site.url}}/assets/images/screen-shot-2015-01-31-at-20-43-54.png)

The deployment will take some time, depending on your storage backend. In my testing lab took around one hour, but it is a nested environment running on NFS so you can expect much better times deploying in a real world setup. When it is finished you can review the different components of VIO with vSphere Web Client in *VMs and Templates*, there would be a new folder structure containing all VIO virtual machines.

[![](/assets/images/image25.png)]({{site.url}}/assets/images/image25.png)

## Validate your VIO installation

In your favorite browser open an HTTPS session against the public hostname or virtual IP address configured during VIO installation. The Horizon portal login page will display.

[![](/assets/images/screen-shot-2015-02-01-at-22-50-41.png)]({{site.url}}/assets/images/screen-shot-2015-02-01-at-22-50-41.png)

Enter the admin credentials and OpenStack admin Overview page will show up. The access the *Hypervisors* area and check that the selected cluster for Nova appears there.

[![](/assets/images/screen-shot-2015-02-01-at-22-53-19.png)]({{site.url}}/assets/images/screen-shot-2015-02-01-at-22-53-19.png)

At this point VIO is setup and you can start to work in Horizon or using the CLI as with any other OpenStack distribution.

**Have fun and happy stacking!**

Juanma.
