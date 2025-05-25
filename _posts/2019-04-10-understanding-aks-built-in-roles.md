---
title: Understanding AKS built-in roles
date: 2019-04-10 09:55:00 +0100
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
- Microsoft
- Containers
- Kubernetes
- Cloud-Native
- AKS
- Azure Kubernetes Service
- DevOps
- Azure
author: juan_manuel_rey
comments: true
---

Every Azure Kubernetes Service cluster comes with two built-in roles in Azure RBAC:

- Azure Kubernetes Service Cluster Admin Role
- Azure Kubernetes Service Cluster User Role

The confusion is that many people tend to believe that these roles are Kubernetes RBAC roles or related to it in some way. I've got the question about this so many times that I decided to write a quick post to clarify what are these roles and the use cases for them.

Both roles are intended exclusively to be used for retrieving credentials with the `az aks get-credentials` command.

- Admin Role allows access to the `Microsoft.ContainerService/managedClusters/listClusterAdminCredential/action` API to get the cluster administrator credentials.
- User Role permits access to the `Microsoft.ContainerService/managedClusters/listClusterUserCredential/action` API and retrieve the cluster user credentials.

In both cases using `az aks get-credentials` command the credentials are merged into a new or an existing `kubeconfig` file. Keep in mind that the user credentials aren't limited to a specific namespace but will have access to all namespaces and will be able to deploy workloads in any of them. To achieve that kind of access you will need to implement RBAC in the cluster which combined with AAD can be very useful to limit the access of your developers and user to their specific namespaces, I will go on the details for this configuration in a future post.

Finally, neither of these roles will by itself allow to perform operations like scale, create or delete an AKS cluster. To be able to perform those operations an Azure user will need to have the Contributor role in the AKS resource group or the whole subscription.

--Juanma
