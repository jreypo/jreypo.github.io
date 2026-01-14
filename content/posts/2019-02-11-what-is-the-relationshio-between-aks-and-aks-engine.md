---
title: What is the relationship between AKS and AKS-Engine?
date: 2019-02-11 13:48:00 +0100
tags:
- aks
- azure
- cloud-native
- containers
- devops
- kubernetes
showComments: true
---

Every time I talk about [AKS](https://azure.microsoft.com/en-us/services/kubernetes-service/) and [AKS-Engine](https://github.com/Azure/aks-engine) got the question about the relationship between both. It is very simple, AKS is backed up by AKS-Engine, plain and simple. It is the library leveraged by the managed service to perform all the creation, upgrade and node scaling of the cluster. It is used the same way as you would use it manually from your shell. So you can consider AKS-Engine like the upstream of AKS.

The below image illustrates it perfectly.

[![](/images/aks-to-aks-engine.png "AKS-Engine relationship to AKS")](/images/aks-to-aks-engine.png)

All the node instances deployed in an AKS cluster has the `aks-engine` version used to deploy it as an Azure Tag. Let's see an example of this using a node from one of my AKS clusters.

Using Azure CLI retrieve the list of nodes of our cluster.

```text
$ az vm list -g MC_aksrg-ready_aks-ready_canadacentral
Name                      ResourceGroup                           Location       Zones
------------------------  --------------------------------------  -------------  -------
aks-nodepool1-68968121-0  MC_aksrg-ready_aks-ready_canadacentral  canadacentral
aks-nodepool1-68968121-1  MC_aksrg-ready_aks-ready_canadacentral  canadacentral
aks-nodepool1-68968121-3  MC_aksrg-ready_aks-ready_canadacentral  canadacentral
```

For our example we will use the first node `aks-nodepool1-68968121-0`. To get the AKS-Engine version used to deploy this cluster we will need to look into the node tags, we can get the nodes by looing the node into the Azure Portal or by using Azure CLI. I will show both ways although I personally prefer the CLI one ;-)

With Azure CLI get the instance Azure Tags.

```text
$ az vm show -n aks-nodepool1-68968121-0 -g MC_AKSRG-READY_AKS-READY_CANADACENTRAL --query '[tags]' -o json
[
  {
    "acsengineVersion": "v0.26.2-aks",
    "creationSource": "acsengine-aks-nodepool1-68968121-0",
    "orchestrator": "Kubernetes:1.11.4",
    "poolName": "nodepool1",
    "resourceNameSuffix": "68968121"
  }
]
```

Take a look at the `acsengineVersion` tag and this will be the AKS-Engine version used to deploy this cluster, in our case is `v0.26.0-aks`.

In the portal go to your virtual machines and select the same node from your AKS cluster and proceed to the Tags section to get the tags assigned to this instance.

[![](/images/aks-engine-version-tag.png "AKS-Engine relationship to AKS")](/images/aks-engine-version-tag.png)

Hope this post helped to clarify a bit the direct relationship between AKS-Engine and AKS.

--Juanma
