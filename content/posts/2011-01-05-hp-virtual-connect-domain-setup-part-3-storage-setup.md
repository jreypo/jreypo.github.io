---
title: 'HP Virtual Connect Domain Setup - Part 3: Storage setup'
date: 2011-01-05
tags:
- hp-servers
- storage
- vmware
showComments: true
image:
  feature: banner-blades.png
---

Welcome to the third post of the [Virtual Connect](http://www.hp.com/go/VirtualConnect) series!

The first two posts were about [initial VC Domain creation]({{< ref "posts/2010-12-20-hp-virtual-connect-domain-setup-part-1-domain-setup-wizard.md" >}} "HP Virtual Connect Domain Setup – Part 1: Domain Setup Wizard") and the [Network setup]({{< ref "posts/2010-12-21-hp-virtual-connect-domain-setup-part-2-network-setup.md" >}} "HP Virtual Connect Domain Setup – Part 2: Network Setup"). In this one I'll explain the storage configuration of the Domain using the **Fibre Channel Setup Wizard**.

Please take into account that **iSCSI** is supported with Virtual Connect since the version 3.10 of Virtual Connect Manager and only with the Flex-10 and FlexFabric modules. However I'm going to leave iSCSI configuration for a future post, since I didn't have many opportunities to try it with VC, and write only about Fibre Channel.

Before we start with the wizard and all the setup task is important to explain the Virtual Connect storage fundamentals.

The first concept to understand are the several key Fibre Channel port types. There a three basic FC ports:

- **N_Port (Node Port)** - An N_Port is a port within a node that provides Fibre Channel attachment like an HBA port. VC-FC module uplink ports are N_ports.
- **F_Port (Fabric Port)** - This a port on a FC switch connected to an N_port and addressable by it. These are commonly used in Edge or Core switches. The VC-FC module's downlink ports are F_ports in order to allow the HBAs to login into them.
- **E_Port (Expansion Port)** - These are switch ports used for switch-to-switch connections known as Inter Switch Link or ISL.

Additionally there are two other ports, however these ports are not typically seen in Virtual Connect environments.

- **NL_Port (Node Loop Port)** - An N_port capable of Arbitrated Loop function.
- **FL_Port (Fabric Loop Port)** - An F_port capable of Arbitrated Loop function.

The next key concept to understand in **N_Port ID Virtualization** or **NPIV**. It's a T11 FC standard than can be defined as a Fibre Channel facility that allows to assign multiple N\_Port\_IDs to a single N_Port, this is a physical N_port having multiple port WWNs. Of course the VC-FC module must be connected to a Fibre Channel switch that supports NPIV.

And how manages Virtual Connect all this port stuff? I believe that an image is worth a thousand words, so first I will show with the below diagrams illustrate how FC ports and SAN are managed with and without Virtual Connect.

[![](/images/blade-san-novc.png "SAN config without VC")](/images/blade-san-novc.png)

As it can be seen the SAN switches, like the [Cisco MDS 9124e](http://h18000.www1.hp.com/storage/saninfrastructure/switches/mds9124e/index.html), that can be used in any blade enclosure including the HP ones are part of the SAN Fabric, that means the enclosure itself is part of the Fabric. These switches are connected to the SAN Core via E_ports or ISL.

[![](/images/hp-vc-fc.png "SAN config with VC")](/images/hp-vc-fc.png)

In this configuration the SAN boundary has been moved out of the enclosure. The VC-FC module includes an HBA Aggregator which is an NPIV device. It passes, transparently, the signals from multiple HBAs to a single switch port.

Here it is how the whole process would go:

1. VC-FC module uplink port issue a Login Request, a *FLOGI* to the SAN and advertise itself as NPIV capable port.
2. Upon receiving an *Accept* from the Fabric it would begin to process server requests.
3. Server HBAs would begin normal Fabric login process with the WWNs.
4. VC-FC module would translate *FLOGI* requests into an *FDISC* requests since a single N\_Port can only receive one *FLOGI* request.
5. SAN switch would reply with an *Accept* and provide HBAs with Fabric addresses.
6. The *Accept* frames would reach uninterrupted the HBAs.
7. From then on all the traffic will be carried over the sane link for all HBA connections.

Now the the basic concepts are explained and, hopefully clear, it's tim to configure the storage.

We are going to use the **Fibre Channel Setup Wizard** to:

- Identify the World Wide Names (WWNs) to be used by the servers.
- Define the available SAN fabrics.

You can launch the wizard either from the Tools menu in the Virtual Connect page or right after finishing the **Network Setup Wizard**. From the welcome screen click Next and move into the **World Wide Name (WWN) Settings** page.

In this first page you can specify if you want to use the WWN settings that comes with the Fiber Channel HBA card or if the HP Virtual Connect supplied WWN settings.

[![](/images/fc_wizard_1.png)](/images/fc_wizard_1.png)

Virtual Connect will assign both a port WWN and a node WWN to a Fibre Channel port, the node WWN will always be the same as the port WWN incremented by one.

There is key advantage when configuring Virtual Connect to assign the WWNs and is that, since it maintains a consistent storage identity, it allows blades to be replaced in case of failure without affecting the external SAN.

In the wizard select **Virtual Connect assigned WWNs** and click **Next** to move into the **Assigned WWNs** screen.

[![](/images/fc_wizard_2.png)](/images/fc_wizard_2.png)

This screen is very similar to the MAC address range selection screen we saw in the previous post. Here you have to choose between an user defined WWNs range and an HP defined one. You must ensure that the selected range is unique within the environment.

Next we are going to define the Fabric, first you'll be presented with a screen asking if you want to define the fabric.

[![](/images/fc_wizard_3.png)](/images/fc_wizard_3.png)

After that we have to enter the Fabric name, assign the uplink ports and configure the speed.

[![](/images/fc_wizard_4.png "fc_")](/images/fc_wizard_4.png)

After applying the configuration the wizard will move to the next screen where it will ask if you want to create more Fabric, for the example purposes I decided to create a another one named *fabric_prod2*.

[![](/images/fc_wizard_6.png)](/images/fc_wizard_6.png)

When you are done with the second fabric finish the wizard and the storage setup will be done. You can review and modify the configuration from the Virtual Connect main interface.

[![](/images/vc-config.png "Virtual Connect main page")]({{images}}/assets/images/vc-config.png)

The next post will be the last of the series and I will discuss about **Virtual Connect Server Profiles**. As always any feedback would be welcome.

Juanma.
