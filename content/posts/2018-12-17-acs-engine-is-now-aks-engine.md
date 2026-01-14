---
title: ACS-Engine is now AKS-Engine
date: 2018-12-17 10:02:00 +0100
categories:
- Containers
- Microsoft
- Cloud-Native
tags:
- Microsoft
- Containers
- Kubernetes
- opensource
- ACS Engine
- AKS Engine
- Azure Container Service
- Azure Kubernetes Service
- AKS
showComments: true
---

This is post is quick heads up for everybody our there using ACS Engine to spin up container clusters on Azure. Our containers engineering team is deprecating `acs-engine` in favor of a new tool called `aks-engine`. The old [ACS-Engine GitHub repo](https://github.com/Azure/acs-engine) will not go away any time soon but from now on all the new developments will be done in the [AKS-Engine](https://github.com/Azure/aks-engine) repo.

In the words of the AKS Engine team *...we're confident this housekeeping maneuver will more effectively track the close affinity between the AKS managed service and the "build and manage your own configurable Kubernetes" stories that folks use this tool for*.

All the Kubernetes related code has been 100% moved with a few exceptions to the new repo but I recommend you to keep an eye open until the transition is complete and of course open an issue in GitHub if you find anything missing.

--Juanma
