---
layout: post
title: Running your private Kubernetes cluster on Azure with ACS Engine
date: 2018-03-19
type: post
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
- Microsoft
- Containers
- Kubernetes
- Cloud-Native
- ACS Engine
- Azure Container Service
author: juan_manuel_rey
comments: true
---

The AKS team has released this week [version v0.14.0](https://github.com/Azure/acs-engine/releases/tag/v0.14.0) of [ACS Engine](https://github.com/Azure/acs-engine). Amongst the many new features and fixed bugs the team has enabled the possibility of creating full private Kubernetes clusters on Azure, meaning that the masters will not be exposed outside of Azure. This will allow for very interesting use cases for Kubernetes on Azure combined with custom VNET deployments and VNET peering, same for those customers with their Azure environments connected to on-prem either with Site to Site VPN or Express Route.  

Without further ado lets deploy our shiny private Kubernetes cluster. First create the cluster definition and on the `kubernetesConfig` api model set `privateCluster` to `true`

```json
      "kubernetesConfig": {
        "privateCluster": {
          "enabled": true
      }}
```

Since this a non-public cluster, to access the masters and execute `kubectl` commands against the API server we will need a jumpbox. The new feature allows to define also the jumpbox if we do not have one or do not want to create it manully. 

```json
      "kubernetesConfig": {
        "privateCluster": {
          "enabled": true,
          "jumpboxProfile": {
            "name": "jumpbox",
            "vmSize": "Standard_D2s_v3",
            "osDiskSizeGB": 30,
            "storageProfile": "ManagedDisks",
            "username": "azureuser",
            "publicKey": "xxx"
          }
      }
```
However right now there is [bug](https://github.com/Azure/acs-engine/issues/2474) on the jumpbox implementation with some missing variables when the deployment is with custom VNET and managed disks configurations, but the [fix is already merged](https://github.com/Azure/acs-engine/pull/2477) on the `acs-engine` master branch and a new release containing this and other fixes will be out sooner than later. For now I would recommend to either use `StorageAccount` for the jumpbox `storageProfile` or provision it manually afterwards. 

My cluster definition file looks like this. 

```json
{
  "apiVersion": "vlabs",
  "properties": {
    "orchestratorProfile": {
      "orchestratorType": "Kubernetes",
      "orchestratorRelease": "1.9",
      "kubernetesConfig": {
        "networkPolicy": "azure",
        "privateCluster": {
          "enabled": true,
          "jumpboxProfile": {
            "name": "jumpbox",
            "vmSize": "Standard_D2s_v3",
            "osDiskSizeGB": 30,
            "storageProfile": "StorageAccount",
            "username": "azuser",
            "publicKey": "<SSH_PUBLIC_KEY>"
          }
        }
      }
    },
    "masterProfile": {
      "count": 1,
      "dnsPrefix": "k8s-19",
      "vmSize": "Standard_D2s_v3"
    },
    "agentPoolProfiles": [
      {
        "name": "agentpool1",
        "count": 3,
        "vmSize": "Standard_D2s_v3",
        "storageProfile": "ManagedDisks",
        "availabilityProfile": "AvailabilitySet"
      }
    ],
    "linuxProfile": {
      "adminUsername": "azuser",
      "ssh": {
        "publicKeys": [
          {
            "keyData": "SSH_PUBLIC_KEY"
          }
        ]
      }
    },
    "servicePrincipalProfile": {
      "clientId": "<AAD_SP_ID>",
      "secret": "<MY_AAD_SP_SECRET>"
      }
    }
}

```

Create the template with `acs-engine`.

```
$ acs-engine generate --api-model k8s-19-private.json
INFO[0000] Generating assets into _output/k8s-19...
$
```

Create the resource group and deploy the cluster. 

```
$ az group create --name k8s-priv-rg -l eastus
Location    Name
----------  -----------
eastus      k8s-priv-rg
$ az group deployment create --name k8s-priv --resource-group k8s-priv-rg --template-file _output/k8s-priv/azuredeploy.json --parameters _output/k8s-priv/azuredeploy.parameters.json --verbose
```

WHen the deployment is finished ssh into the jumpbox, `kubectl` should be automatically configured, and get the cluster info. 

```
azuser@jumpbox:~$ kubectl cluster-info
Kubernetes master is running at https://10.255.255.5
Heapster is running at https://10.255.255.5/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at https://10.255.255.5/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://10.255.255.5/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://10.255.255.5/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
tiller-deploy is running at https://10.255.255.5/api/v1/namespaces/kube-system/services/tiller-deploy:tiller/proxy
```

You can see that our Kubernetes master is exposed on a private address instead of being on the internet. 

As I said, this opens many new possibilities for Kubernetes on Azure. Comments are welcome. 

-- Juanma