---
title: Using Azure CLI to get info about your AKS cluster
date: 2019-04-09 11:10:00 +0100
categories:
- Containers
- Microsoft
- Azure
- Cloud
- Cloud-Native
- DevOps
tags:
- Microsoft
- Containers
- Kubernetes
- Cloud-Native
- AKS-Engine
- Azure Kubernetes Service
- DevOps
- Azure
- Azure CLI
showComments: true
---

As Kubernetes admins or users using `kubectl` is the most common way we have to interact with our clusters, you can deploy workloads, get info from your clusters, troubleshoot and large array of tasks, etc. However if your Kubernetes cluster is running on Azure as an AKS managed cluster using Azure CLI you can get a lot of useful information about the state of the service and the different Azure resources that make up your cluster. In this post I will show you some examples that can be helpful.

This can be achieved thanks to Azure CLI `--query` argument that will allow us to execute a [JMESPath](http://jmespath.org/) query on the output of an Azure CLI command. I strongly recommend to check the [Azure CLI documentation](https://docs.microsoft.com/en-us/cli/azure/query-azure-cli?view=azure-cli-latest) for more details on how to use it and some useful examples.

The best starting point is to retrieve all the data about the cluster in JSON format.

```text
$ az aks show -n aks-demo -g k8s-demo-rg -o json
{
  "aadProfile": null,
  "addonProfiles": {
    "aciConnectorLinux": {
      "config": {
        "SubnetName": "aks-vn-subnet"
      },
      "enabled": true
    },
    "httpApplicationRouting": {
      "config": {
        "HTTPApplicationRoutingZoneName": "10434ea34ba446eebb68.westeurope.aksapp.io"
      },
      "enabled": true
    },
    "omsagent": {
      "config": {
        "logAnalyticsWorkspaceResourceID": "/subscriptions/de39160d-7900-4026-a806-36dd5325af60/resourcegroups/mms-weu/providers/microsoft.operationalinsights/workspaces/starlabs"
      },
      "enabled": true
    }
  },
  "agentPoolProfiles": [
    {
      "count": 3,
      "maxPods": 30,
      "name": "nodepool1",
      "osDiskSizeGb": 30,
      "osType": "Linux",
      "storageProfile": "ManagedDisks",
      "vmSize": "Standard_DS2_v2",
      "vnetSubnetId": "/subscriptions/de39160d-7900-4026-a806-36dd5325af60/resourceGroups/k8s-demo-rg/providers/Microsoft.Network/virtualNetworks/aks-demo-vnet/subnets/aks-subnet-1"
    }
  ],
  "dnsPrefix": "aks-demo",
  "enableRbac": true,
  "fqdn": "aks-demo-60a7ad35.hcp.westeurope.azmk8s.io",
  "id": "/subscriptions/de39160d-7900-4026-a806-36dd5325af60/resourcegroups/k8s-demo-rg/providers/Microsoft.ContainerService/managedClusters/aks-demo",
  "kubernetesVersion": "1.11.5",
  "linuxProfile": {
    "adminUsername": "azuser",
    "ssh": {
      "publicKeys": [
        {
          "keyData": "XXXXXX"
        }
      ]
    }
  },
  "location": "westeurope",
  "name": "aks-demo",
  "networkProfile": {
    "dnsServiceIp": "10.0.0.10",
    "dockerBridgeCidr": "172.17.0.1/16",
    "networkPlugin": "azure",
    "networkPolicy": null,
    "podCidr": null,
    "serviceCidr": "10.0.0.0/16"
  },
  "nodeResourceGroup": "MC_k8s-demo-rg_aks-demo_westeurope",
  "provisioningState": "Succeeded",
  "resourceGroup": "k8s-demo-rg",
  "servicePrincipalProfile": {
    "clientId": "xxxxxxx"
  },
  "type": "Microsoft.ContainerService/ManagedClusters"
}
```

With the previous command we will retrieve all the information from an AKS cluster but we can also get specific data like all the configuration values for the agent pool in our cluster.

```text
$ az aks show -n aks-demo -g k8s-demo-rg --query agentPoolProfiles -o json
[
  {
    "count": 3,
    "maxPods": 30,
    "name": "nodepool1",
    "osDiskSizeGb": 30,
    "osType": "Linux",
    "storageProfile": "ManagedDisks",
    "vmSize": "Standard_DS2_v2",
    "vnetSubnetId": "/subscriptions/de39160d-7900-4026-a806-36dd5325af60/resourceGroups/k8s-demo-rg/providers/Microsoft.Network/virtualNetworks/aks-demo-vnet/subnets/aks-subnet-1"
  }
]
```

Or we can get a specific value like the maximum number of pods per node allowed in this cluster.

```text
$ az aks show -n aks-demo -g k8s-demo-rg --query '[{MaxPods:agentPoolProfiles[0].maxPods}]' -o table
MaxPods
---------
30
```

We can get the cluster resource ID as well.

```text
$ az aks show -n aks-demo -g k8s-demo-rg -o json --query id
"/subscriptions/de39160d-7900-4026-a806-36dd5325af60/resourcegroups/k8s-demo-rg/providers/Microsoft.ContainerService/managedClusters/aks-demo"
```

A query I find particularly useful is to get the network profile configured in the cluster, shown below in JSON and table output format.

```text
$ az aks show -n aks-demo -g k8s-demo-rg --query networkProfile -o table
NetworkPlugin    ServiceCidr    DnsServiceIp    DockerBridgeCidr
---------------  -------------  --------------  ------------------
azure            10.0.0.0/16    10.0.0.10       172.17.0.1/16
$
$ az aks show -n aks-demo -g k8s-demo-rg --query networkProfile -o json
{
  "dnsServiceIp": "10.0.0.10",
  "dockerBridgeCidr": "172.17.0.1/16",
  "networkPlugin": "azure",
  "networkPolicy": null,
  "podCidr": null,
  "serviceCidr": "10.0.0.0/16"
}

```

As you probably know every AKS cluster is tied to two different resource groups in Azure, one for cluster object itself and another one where the Kubernetes nodes and all the related resources are deployed. You can easily get its name by querying the AKS cluster for the `nodeResourceGroup`. This will be useful to perform queries about the nodes that form our cluster.

```text
$ az aks show -n aks-demo -g k8s-demo-rg --query nodeResourceGroup
Result
----------------------------------
MC_k8s-demo-rg_aks-demo_westeurope
```

Finally we are going to get the available versions for an upgrade operation in our cluster.

```text
$ az aks get-upgrades -n aks-demo -g k8s-demo-rg
Name     ResourceGroup    MasterVersion    NodePoolVersion    Upgrades
-------  ---------------  ---------------  -----------------  ------------------------------
default  k8s-demo-rg      1.11.5           1.11.5             1.11.8, 1.11.9, 1.12.6, 1.12.7
```

Besides of the cluster, the Kubernetes nodes or to be more precise the Azure virtual machines were the node agent software is running, can also be queried to get some useful data. I already showed in a previous [post about the relationship between AKS and AKS-Engine]({{< ref "posts/2019-02-11-what-is-the-relationshio-between-aks-and-aks-engine.md" >}}) how to check for the `aks-engine` version used to deploy a cluster from the Azure tags of any of the nodes.

```text
$ az vm show -n aks-nodepool1-41320097-0 -g MC_k8s-demo-rg_aks-demo_westeurope --query '[tags.acsengineVersion]'
Result
-----------
v0.26.0-aks
```

Or we can list all the tags for the node.

```text
$ az vm show -n aks-nodepool1-41320097-0 -g MC_k8s-demo-rg_aks-demo_westeurope -o json --query tags
{
  "acsengineVersion": "v0.26.0-aks",
  "creationSource": "aks-aks-nodepool1-41320097-0",
  "orchestrator": "Kubernetes:1.11.5",
  "poolName": "nodepool1",
  "resourceNameSuffix": "41320097"
}
```

Hope this post has been helpful, I wanted to use it not only to show how to get information from an AKS cluster but also as a good example of how powerful Azure CLI queries are.

--Juanma
