---
title: A first look into Azure VMware Solution
date: 2021-02-19 15:00:00 +0100
type: post
published: true
status: publish
categories:
- Microsoft
- VMware
- Cloud
tags:
- Microsoft
- AVS
- Azure
- VMware
- Azure VMware Solution
- Cloud
author: juan_manuel_rey
comments: true
---

As I said in my previous post I moved last year to Microsoft Azure engineering in the Azure VMware Solution product group, so it makes total sense that my first post in the new era of the blog is about AVS. Let's begin!

# What is Azure VMware Solution?

Well, that's easy. Azure VMware Solution, or AVS, is a first-party Azure service that allows customers to run native VMware workloads on Azure. The important part if **first-party**, because **AVS is Azure**, it not a third party service or a partner delivered service, Microsoft operates and supports the service which has been built in collaboration with VMware. It provides the customer with a vSphere-based Private Cloud, built on dedicated hardware on an Azure region.

All provisioned private clouds have vCenter Server, vSAN, vSphere, and NSX-T. You can migrate workloads from your on-premises environments, deploy new virtual machines (VMs), and consume Azure services from your private clouds. VMware HCX Advanced is also provided in the AVS software stack to enable the workload migration scenarios and limited disaster recovery

Today the service is available in East US, West US, North Central US, Canada Central, UK South, West Europe, Japan East and Australia East regions, with more Azure regions coming in the near future.

## AvS Service Components

AVS comes with the following VMware products bundled and licensed, no need for customer to buy additional licenses separately fromm VMware.

- vSphere 6.7 Update 3 Enterprise Plus
- VSAN 6.7 Enterprise
- NSX-T 2.5.2
- HCX R139 Advanced

On the hardware side AV36 is the only SKU available today to deploy your AVS Private Cloud.

- **CPU** - Intel Xeon Gold 6140 2.3 GHz
- **Memory** - 576 GB
- **Storage vSAN Caching** - 2 × 1.6 TB NVMe
- **Storage vSAN Capacity** - 8 × 1.92 TB SSD
- **Network** - 2 Dual Port 25 GbE

## AVS Architecture

As seen in the previous section AVS is built on top of VMware Cloud Foundation, deployed on dedicated, bare-metal Azure hosts. The architecture of the service is more or less like this one.

[![](/assets/images/avs_architecture.png)]({{site.url}}/images//avs_architecture.png)

To enable the connectivity between AVS workloads and the main Azure fabric [ExpressRoute Gloabl Reach](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-global-reach) is used. ExpressRoute is a dedicated line that enables customers to connect their on-premises environment into Azure and Global Reach is an ExpressRoute add-on that allows to link ExpressRoute circuits together to make a private network between customer on-premises networks, in this case is used to link AVS ExpressRoute circuit with an existing customer circuit. Since transitive routing between circuits is not enabled on Azure ExpressRoute Gateways, the usage of Global Reach is mandatory in order to interconnect an on-premises vSphere environment and AVS.

ExpressRoute Global Reach is needed as well for VMware HCX since it is not supported over an Azure Site-to-Site VPN connection.

[Azure Virtual WAN](https://docs.microsoft.com/en-gb/azure/virtual-wan/) acts as a communications hub between on-premises and Azure IaaS and PaaS services, for AVS running workloads Azure VWAN will provide Public IP capabilities through its integrated Azure Firewall.

# Getting started with the service

Getting started with AVS service requires for your Azure subscription to be whitelisted and get quota assigned, follow instructions on [AVS documentation](https://docs.microsoft.com/en-us/azure/azure-vmware/enable-azure-vmware-solution). With the quota assigned register the Azure VMware Solution resource provider using Azure CLI.

```
az provider register -n Microsoft.AVS --subscription <your subscription ID>
```

Check the registration state until it appears as `Registered`.

```
$ az provider show -n Microsoft.AVS
Namespace      RegistrationPolicy    RegistrationState
-------------  --------------------  -------------------
Microsoft.AVS  RegistrationRequired  Registered
```

Once the provider is registered go to [Azure Portal](https://portal.azure.com), on the main screen click on **Create a Resource** and search for Azure VMware Solution to deploy a new Private Cloud.

[![](/assets/images/avs-create-screen.png)]({{site.url}}/images/avs-create-screen.png)

In the above screen the following parameters are needed.

- **Resource Group** – The Azure resource group that will contain he AVS Private Cloud resource.
- **Location** – Azure region to deploy the new Private Cloud.
- **Resource Name** – Name for the Private Cloud.
- **SKU** – Node type to deploy, currently there is only one **AV36**.
- **ESXi Hosts** – Number of hosts to deploy, the minimum number is 3 and can be later scale out up to a max of 16 per cluster and a maximum of 4 clusters.
- **vCenter admin Password** – Password for `cloudadmin@vsphere.local` user to access vCenter Server. Full admin rights are not granted for vCenter.
- **NSX-T Manager Password** – Password for NSX-T admin user.
- **Address Block** – CIDR block used when deploying management components, requires a /22 segment and only [RFC 1918](https://tools.ietf.org/html/rfc1918) private address spaces are permitted. See table below using 172.16.0.0/22 as CIDR.
- **Virtual Network** – Select an existing VNET or create a new one, this VNET will be later used to deploy an ExpressRoute Gateway and a jumpbox to be able to access the Private Cloud management components.

| Network usage             | Subnet            |
| ------------------------- | ----------------- |
| Private cloud management  | `172.16.0.0/26`   |
| HCX Mgmt Migrations       | `172.16.0.64/26`  |
| Global Reach Reserved     | `172.16.0.128/26` |
| ExpressRoute Reserved     | `172.16.0.192/27` |
| ExpressRoute peering      | `172.16.1.0/25`   |
| vMotion Network           | `172.16.1.128/25` |
| Replication Network       | `172.16.2.0/25`   |
| vSAN                      | `172.16.2.128/25` |
| HCX Uplink                | `172.16.3.0/26`   |
| Reserved                  | `172.16.3.64/26`  |
| Reserved                  | `172.16.3.128/26` |
| Reserved                  | `172.16.3.192/26` |

After filling in all the required parameters launch the private cloud creation. The deployment wll take a couple of hours to complete.

Once the deployment is complete the quickest way to access your private cloud would be trough a jumpbox, deploy a Windows virtual machine connected to a subnet of the selected or created VNET. RDP into the jumpbox or even better use [Azure Bastion](https://azure.microsoft.com/en-us/services/azure-bastion/) too to avoid exposing the VM publicly.

vCenter Server and NSX-T Managers URLs and credentials can be retrieved from Azure portal in the **Identity** blade of the AVS private cloud.

[![](/assets/images/avs-identity.png)]({{site.url}}/images/avs-identity.png)

From the desktop of the jumpbox access vCenter Server and NSX-T Manager to verify that everything is up and runnning.

[![](/assets/images/avs-bastion.png)]({{site.url}}/images/avs-bastion.png)

At this point the next steps will be configure NSX-T DHCP and network segments and deploy a virtual machine, i encourage you to review Azure VMware Solution documentation for the details. I will go more deep into NSX-T and general network architecture in AVS in a future post.

Thanks for staying to the end and again please stay safe out there.

--Juanma
