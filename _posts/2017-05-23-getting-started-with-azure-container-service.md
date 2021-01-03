---
title: Getting started with Azure Container Service
date: 2017-05-23
type: post
classes: wide
published: true
status: publish
categories:
- Containers
- Microsoft
- Azure
- Cloud
- Cloud-Native
tags:
- Docker
- Azure
- ACS
- Azure Container Service
- Containers
- Kubernetes
- Mesos
- DC/OS
- Mesosphere
- Microsoft
- Cloud-Native
author: juan_manuel_rey
comments: true
---

**Azure Container Service**, or ACS, is the container management service in Microsoft Azure cloud. The main idea behind ACS is to create an optimized container hosting infrastructure on top of Microsoft Azure and let the system engineers and developers focus on managing their fleet of containerized applications using common, standard open source tools.

ACS provides an easy way to automate the creation and management of groups of virtual machines pre-configured as container orchestration clusters. ACS deploys all the underlaying infrastructure by leveraging Azure platform capabilities like Azure Resource Manager, Networking, Security and Virtual Machine Scale Sets. ACS also deploys the selected container orchestration software. 

Currently ACS supports the three most popular container orchestrators in the industry.

- [Docker Swarm](https://docs.docker.com/swarm/)
- [Mesosphere DC/OS](https://dcos.io/)
- [Kubernetes](https://kubernetes.io/)

# ACS-engine

The core of Azure Container Service is [acs-engine](https://github.com/azure/acs-engine). An opensource project from Microsoft written in [Go](https://golang.org/), `acs-engine` by itself allows you te generate [ARM](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview) templates with customized configurations for the three different orchestrators. For example with `acs-engine` you can generate an ARM template to deploy a Swarmkit Docker cluster or a cluster with a custom VNet, there are multiple possibilities and in future posts I will show a couple of these examples. 

There is one caveat, any cluster deployed with `acs-engine` will not show as an ACS cluster in Azure since currently `acs-engine` is not a [resource provider](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#resource-providers) in ARM.

# Deploy your first ACS cluster

For this first post about ACS we will deploy a simple cluster with the default options and DC/OS orchestrator, which is the default orchestrator for ACS. I will use [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/overview) but you can use [Azure PowerShell](https://docs.microsoft.com/en-us/powershell/azure/overview?view=azurermps-4.0.0) or the [Azure Portal](https://portal.azure.com). 

First create a new resource group, is not mandatory since you can always create your cluster under an existing resource group however in my experience is better to deploy your ACS clusters each on its own group. To create the group you have to provide the name of the group and the Azure region where it will be created, in my case Western Europe. 

```
$ az group create -n acsrg1 -l westeurope
Location    Name
----------  ------
westeurope  acsrg1
```

Next create the cluster, the mandatory parameters are the name for the cluster, the resource group and a DNS prefix. You will need a pregenerated SSH key in order to access the master(s) but Azure CLI can generate it for you by appending `--generate-ssh-keys`.

```
az acs create -g acsrg1 -n acs-dcos-1 -d dcos-cluster-1 --verbose
```

It is as simple as it looks, ACS will deploy and configure for you the DC/OS cluster. However this is a default deployment, you can customize many of the cluster parameters like number of agents, number of masters, size of the VMs, etc. By defual tit will select the following ones:

- One master.
- Three agents.
- Virtual machine size: Standard_DS2_v2.
- Admin user: azureuser.

It will create as well a Service Principal, which will allow the cluster to provision resources on Azure within your subscription, like Azure Storage and Load Balancers. For my tests and labs I personally use a smaller configuration, have my own SSH keys and a pre-created Service Principal. 

```
az acs create -g acsrg2 -n acs-dcos-2 -d dcos-cluster-2 --master-count 1 --agent-count 1 --service-principal 11111111-1111-1111-1111-111111111111 --client-secret=xxxxxxxxx --verbose
```

# Exploring the cluster

After the succesful creation we will use Azure CLI to list and manage the existing clusters and the different resources on each of them. 

```
$ az acs list
Location    Name         ProvisioningState    ResourceGroup
----------  -----------  -------------------  ---------------
westeurope  acs-dcos-1   Succeeded            ACSRG1
westeurope  acs-dcos-2   Succeeded            ACSRG2
westeurope  swarm-acs-1  Succeeded            ACSSWARMRG1
westeurope  k8s-acs-2    Succeeded            K8SACSRG2
```

A DC/OS ACS cluster contain manye different Azure resources:

- Master instance(s)
- Public agent(s) - Up to three depending on the number of private agents. 
- Private agent(s)
- Storage
- Networks
- Load Balancers

For DC/OS the deployed architecture can be seen in the below diagram:

[![](/assets/images/dcos_architecture.png "DC/OS Architecture")]({{site.url}}/assets/images/dcos_architecture.png)

The agents are organized in two different pools, a public pool and a private pool. You workloads can be deployed to either of theses pools, however there will be differences in the accesibility of the instances since public agents are exposed to the internet through an Azure Load Balancer and private agents will be kept internal. These are basic [DC/OS security concepts](https://dcos.io/docs/1.7/administration/securing-your-cluster/) and are not imposed by Azure Container Service. 

- Private Agents: Private nodes are run through a non-routable network, accesible only from the Admin zone or through a public edge router. By default DC/OS will deploy the workloads on private agent nodes.
- Public Agents: Public agents are used to run apps on DC/OS through a publicly accesible network. 

For Kubernetes and Docker Swarm the concept of private agent does not apply. 

## Master instances

The masters will show up as independent virtual machines.

```
$ az vm list
Name                      ResourceGroup    Location
------------------------  ---------------  ----------
dcos-master-633E21A3-0    ACSRG1           westeurope
dcos-master-A59D4620-0    ACSRG2           westeurope
swarm-master-D0CD9D3-0    ACSSWARMRG1      westeurope
docker-01                 COREOSRG         westeurope
k8s-agent-787682B6-0      K8SACSRG2        westeurope
k8s-agent-787682B6-1      K8SACSRG2        westeurope
k8s-agent-787682B6-2      K8SACSRG2        westeurope
k8s-master-787682B6-0     K8SACSRG2        westeurope
swarmm-master-71393939-0  SWARMRG1         westeurope
testvm                    TESTRG           westeurope
```
The masters can be accessed through SSH using ssh-key authentication with the default user, `azureuser`, or a custom user defined durign the cluster creation. To get he master piblic FQDN you can use the Azure Portal or Azure CLI by retrieving the configuration of the cluster in JSON format.

```
az acs show -n acs-dcos-1 -g acsrg1  -o json
```

The JSON output should be like the one below.

```json
{
  "agentPoolProfiles": [
    {
      "count": 3,
      "dnsPrefix": "dcos-cluster-1agents",
      "fqdn": "dcos-cluster-1agents.westeurope.cloudapp.azure.com",
      "name": "agentpools",
      "vmSize": "Standard_D2_v2"
    }
  ],
  "customProfile": null,
  "diagnosticsProfile": {
    "vmDiagnostics": {
      "enabled": true,
      "storageUri": "https://xxxxxx.blob.core.windows.net/"
    }
  },
  "id": "/subscriptions/11111111-1111-1111-1111-1111111111111/resourceGroups/acsrg1/providers/Microsoft.ContainerService/containerServices/acs-dcos-1",
  "linuxProfile": {
    "adminUsername": "azureuser",
    "ssh": {
      "publicKeys": [
        {
          "keyData": "ssh-rsa xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        }
      ]
    }
  },
  "location": "westeurope",
  "masterProfile": {
    "count": 1,
    "dnsPrefix": "dcos-cluster-1mgmt",
    "fqdn": "dcos-cluster-1mgmt.westeurope.cloudapp.azure.com"
  },
  "name": "acs-dcos-1",
  "orchestratorProfile": {
    "orchestratorType": "DCOS"
  },
  "provisioningState": "Succeeded",
  "resourceGroup": "acsrg1",
  "servicePrincipalProfile": null,
  "tags": null,
  "type": "Microsoft.ContainerService/ContainerServices",
  "windowsProfile": null
}
```
We need the `fqdn` parameter from `masterProfile`.

DC/OS graphic interface can be accessed by creating an SSH tunnel to the master. 

```
sudo ssh -fNL 80:localhost:80 -p 2200 -i /home/jurey/.ssh/id_rsa azureuser@dcos-cluster-1mgmt.westeurope.cloudapp.azure.com
```

After the tunnel is stablished point your local browser to the following URLs.

- **DC/OS UI** - https://localhost:80

[![](/assets/images/dcos_ui.png "DC/OS web interface")]({{site.url}}/assets/images/dcos_ui.png)

- **Marathon** - http://localhost:80/marathon 

[![](/assets/images/marathon_ui.png "Marathon web console")]({{site.url}}/assets/images/marathon_ui.png)

- **Mesos** - http://localhost:80/mesos

[![](/assets/images/mesos_ui.png "Mesos web interface")]({{site.url}}/assets/images/mesos_ui.png)

## Agent instances

The agent pools are deployed inside two VM Scale Sets, one of the private and one for the public ones. A VM Scale Set is an Azure compute resource that groups set of identical virtual machines to be jointly managed, this enables auto-scaling capabilities with no need of pre-provisioning any virtual machine. 

```
$ az vmss list
Location    Name                               Overprovision    ProvisioningState    ResourceGroup    SinglePlacementGroup
----------  ---------------------------------  ---------------  -------------------  ---------------  ----------------------
westeurope  dcos-agent-private-633E21A3-vmss0  True             Succeeded            ACSRG1           True
westeurope  dcos-agent-public-633E21A3-vmss0   True             Succeeded            ACSRG1           True
westeurope  dcos-agent-private-A59D4620-vmss0  True             Succeeded            ACSRG2           True
westeurope  dcos-agent-public-A59D4620-vmss0   True             Succeeded            ACSRG2           True
westeurope  swarm-agent-D0CD9D3-vmss           True             Succeeded            ACSSWARMRG1      True
westeurope  swarmm-agentpublic-71393939-vmss   True             Succeeded            SWARMRG1         True
```

As you can see Docker Swarm and DC/OS use VM Scale Sets for the agent instances but as I will explain in the next post of the ACS series currently Kubernetes does not.

We can list and inspect the instances within any VM Scale Set, in our case we'll have a look into the private and public ones for acs-dcos-1 cluster. 

```
$ az vmss list-instances -n dcos-agent-private-633E21A3-vmss0 -g acsrg1
  InstanceId  LatestModelApplied    Location    Name                                 ProvisioningState    ResourceGroup    VmId
------------  --------------------  ----------  -----------------------------------  -------------------  ---------------  ------------------------------------
           0  True                  westeurope  dcos-agent-private-633E21A3-vmss0_0  Succeeded            ACSRG1           fb9a10b9-d8cc-46ae-bfd0-9cebaa39bf1b
           3  True                  westeurope  dcos-agent-private-633E21A3-vmss0_3  Succeeded            ACSRG1           fda533bf-4676-4ab7-9008-9219bdbe7e1f
           4  True                  westeurope  dcos-agent-private-633E21A3-vmss0_4  Succeeded            ACSRG1           8e0c9bdb-d32c-4a9e-91ff-5b8c8f11ced2
$ az vmss list-instances -n dcos-agent-public-633E21A3-vmss0 -g acsrg1
  InstanceId  LatestModelApplied    Location    Name                                ProvisioningState    ResourceGroup    VmId
------------  --------------------  ----------  ----------------------------------  -------------------  ---------------  ------------------------------------
           1  True                  westeurope  dcos-agent-public-633E21A3-vmss0_1  Succeeded            ACSRG1           a9b0dfdf-f4f6-4b7c-ac12-e8de7a39da76
```

### Scaling the agents

Scaling up or down an ACS cluster is a very straightforward task that can be easily perform with Azure CLI.

```
az acs scale -n acs-dcos-1 -g acsrg1 --new-agent-count 6
```

You can monitor the task by looking at the VM Scale Set. When the scale operation is finished the `ProvisioningState` of every node must be `Succeeded`.

```
$ az vmss list-instances -n dcos-agent-private-633E21A3-vmss0 -g acsrg1
  InstanceId  LatestModelApplied    Location    Name                                  ProvisioningState    ResourceGroup    VmId
------------  --------------------  ----------  ------------------------------------  -------------------  ---------------  ------------------------------------
           0  True                  westeurope  dcos-agent-private-633E21A3-vmss0_0   Succeeded            ACSRG1           fb9a10b9-d8cc-46ae-bfd0-9cebaa39bf1b
           3  True                  westeurope  dcos-agent-private-633E21A3-vmss0_3   Succeeded            ACSRG1           fda533bf-4676-4ab7-9008-9219bdbe7e1f
           4  True                  westeurope  dcos-agent-private-633E21A3-vmss0_4   Succeeded            ACSRG1           8e0c9bdb-d32c-4a9e-91ff-5b8c8f11ced2
           7  True                  westeurope  dcos-agent-private-633E21A3-vmss0_7   Creating             ACSRG1           3f4d5be5-1107-414d-8769-b61d55a852aa
           8  True                  westeurope  dcos-agent-private-633E21A3-vmss0_8   Creating             ACSRG1           44174b24-f93f-4677-9933-16cea57f9d5a
          10  True                  westeurope  dcos-agent-private-633E21A3-vmss0_10  Creating             ACSRG1           04c762ed-4af2-44ae-a056-f6ddf31c3860
```

## Networking resources

For each cluster ACS will deploy the needed networking constructs, right no this is not customizable with ACS but it can be achieved using `acs-engine`. Let's see the network resources for the acs-dcos-1 cluster.

```
$ az resource list -g acsrg1 | grep Microsoft.Network
dcos-agent-lb-633E21A3                       acsrg1           westeurope  Microsoft.Network/loadBalancers
dcos-master-lb-633E21A3                      acsrg1           westeurope  Microsoft.Network/loadBalancers
dcos-master-633E21A3-nic-0                   acsrg1           westeurope  Microsoft.Network/networkInterfaces
dcos-agent-private-nsg-633E21A3              acsrg1           westeurope  Microsoft.Network/networkSecurityGroups
dcos-agent-public-nsg-633E21A3               acsrg1           westeurope  Microsoft.Network/networkSecurityGroups
dcos-master-nsg-633E21A3                     acsrg1           westeurope  Microsoft.Network/networkSecurityGroups
dcos-agent-ip-dcos-cluster-1agents-633E21A3  acsrg1           westeurope  Microsoft.Network/publicIPAddresses
dcos-master-ip-dcos-cluster-1mgmt-633E21A3   acsrg1           westeurope  Microsoft.Network/publicIPAddresses
dcos-vnet-633E21A3                           acsrg1           westeurope  Microsoft.Network/virtualNetworks
```

As it can be seen there is a VNet, `dcos-vnet-633E21A3`, that has three subnets created within it address space:
- Master subnet
- Private agent subnet
- Public agent subnet

```
$ az network vnet show -n dcos-vnet-633E21A3 -g acsrg1
Location    Name                ProvisioningState    ResourceGroup    ResourceGuid
----------  ------------------  -------------------  ---------------  ------------------------------------
westeurope  dcos-vnet-633E21A3  Succeeded            acsrg1           8548d496-4e95-4eb2-9b0a-ab7742ea89c1
$ az network vnet show -n dcos-vnet-633E21A3 -g acsrg1 -o json | grep -A3 addressSpace
  "addressSpace": {
    "addressPrefixes": [
      "172.16.0.0/24",
      "10.0.0.0/8"
$ az network vnet subnet list --vnet-name dcos-vnet-633E21A3 -g acsrg1
AddressPrefix    Name                     ProvisioningState    ResourceGroup
---------------  -----------------------  -------------------  ---------------
172.16.0.0/24    dcos-masterSubnet        Succeeded            acsrg1
10.0.0.0/11      dcos-agentPublicSubnet   Succeeded            acsrg1
10.32.0.0/11     dcos-agentPrivateSubnet  Succeeded            acsrg1
```

There are also two load balancers with two public IP addresses, one for each. One load balancer is for the master and the other for the agents. 

```
$ az network lb list -g acsrg1
Location    Name                     ProvisioningState    ResourceGroup    ResourceGuid
----------  -----------------------  -------------------  ---------------  ------------------------------------
westeurope  dcos-agent-lb-633E21A3   Succeeded            acsrg1           23d1227e-c754-4491-b034-4ccd37956951
westeurope  dcos-master-lb-633E21A3  Succeeded            acsrg1           b9194bf6-0e1d-4b47-af05-c6873fbedf22
$ az network public-ip list -g acsrg1
  IdleTimeoutInMinutes  IpAddress      Location    Name                                         ProvisioningState    PublicIpAddressVersion    PublicIpAllocationMethod    ResourceGroup    ResourceGuid
----------------------  -------------  ----------  -------------------------------------------  -------------------  ------------------------  --------------------------  ---------------  ------------------------------------
                     4  xx.xx.xx.xx    westeurope  dcos-agent-ip-dcos-cluster-1agents-633E21A3  Succeeded            IPv4                      Dynamic                     acsrg1           3d7879b7-194f-41d3-9153-5c0292a3771a
                     4  xx.xx.xx.xx    westeurope  dcos-master-ip-dcos-cluster-1mgmt-633E21A3   Succeeded            IPv4                      Dynamic                     acsrg1           3559ad40-31e1-43a6-9905-9c1b4c656d92
```

In the next post about Azure Container Service I will start with the creation and management of Kubernetes clusters on ACS, which happens to be my favorite container orchestrator :D

Finally if you want to see some really cool stuff with Azure, ACS, DC/OS and vSphere check my friend Lior series of posts **Mesosphere DCOS, Azure, Docker, VMware & Everything Between**, [Part 1](http://imallvirtual.com/dcos-azure-docker-vmware-everything-part-1/), [Part 2](http://imallvirtual.com/mesosphere-dcos-azure-docker-vmware-everything-part-2/), [Part 3](http://imallvirtual.com/mesosphere-dcos-azure-docker-vmware-everything-part-3/) and [Part 4](http://imallvirtual.com/mesosphere-dcos-azure-docker-vmware-everything-part-4/) with more parts coming soon. 

--Juanma