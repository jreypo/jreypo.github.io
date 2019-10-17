---
layout: post
title: AKS integration with Azure Container Registry
date: 2019-10-16 15:28:00 +0100
type: post
published: true
status: publish
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
- AKS
- AKS
- Azure Container Registry
- Azure Kubernetes Services
- DevOps
- Azure
author: juan_manuel_rey
comments: true
---

Since the beginning of the service AKS has been able to use Azure Container Registry, or ACR, to pull the container images used in a deployment initiated either by an engineer or by a CD pipeline. However very recently the team shipped a new tight integration between AKS and ACR. 

You must have at least Azure CLI 2.0.73 installed on you laptop or use Azure Cloud Shell.

# Integrate ACR during AKS cluster creation

This new integration allows for an easy setup of the authentication mechanism during cluster creation and easily enabling it for an existing cluster as well. During the creation operation is as simple as use the `--attach-acr` option with the regitry name as the parameter.

```
az aks create -n aks-cl1 -g k8s-demo-rg2 --dns-name-prefix aks-cl1 --admin-username azuser -l westeurope --attach-acr acr-demo-1
```

# Integrate ACR with an existing AKS cluster

A common scenario would be to have an already existing ACR and AKS cluster that you want to have integrated. To do so we must perform an `update` operation with the same `--attach-acr` option.

```
$ az aks update --name aks-demo2 --resource-group aksdemo2-rg --attach-acr acr-demo-1
AAD role propagation done[############################################]  100.0000%┌
$
```

--Juanma
