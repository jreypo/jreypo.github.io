---
layout: post
title: Kubernetes version upgrade with AKS-Engine
date: 2019-02-07 19:45:00 +0100
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
- Microsoft
- Containers
- Kubernetes
- Cloud-Native
- AKS-Engine
- Azure Kubernetes Service
- DevOps
- Azure
author: juan_manuel_rey
comments: true
---

Performing a version upgrade in [Kubernetes](https://kubernetes.io/) can be a challenging task. With [AKS](https://azure.microsoft.com/en-us/services/container-service/), and the likes, you have that resolved and solutions like [Red Hat OpenShift](https://www.openshift.com/) also provide a set of tools for this the upgrade operation.. However what happens if you have deployed your Kubernetes cluster on Azure using [AKS-Engine](https://github.com/Azure/aks-engine)? Well, fortunately for you AKS-Engine has you covered. 

Before performing any operation, upgrade or any other, I strongly recommend you to download and use the latest `aks-engine` version, I've found issues in the upgrade operation wiht older versions. In my case I am using version 0.29.1.

```
$ aks-engine version
Version: v0.29.1
GitCommit: b35549b
GitTreeState: clean
```

To demostrate the upgrade procedure I provioned a 1.11.5 cluster with one master in an Azure Availabbility Set and three nodes in a Virtual Machine Scale Set. To upgrade the Kubernetes version first get the available versions to upgrade to. 

```
$ kubectl get nodes -o wide
NAME                                 STATUS    ROLES     AGE       VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k8s-agentpool1-44862260-vmss000000   Ready     agent     3m        v1.11.5   10.240.0.66    <none>        Ubuntu 16.04.5 LTS   4.15.0-1035-azure   docker://3.0.1
k8s-agentpool1-44862260-vmss000001   Ready     agent     3m        v1.11.5   10.240.0.35    <none>        Ubuntu 16.04.5 LTS   4.15.0-1035-azure   docker://3.0.1
k8s-agentpool1-44862260-vmss000002   Ready     agent     3m        v1.11.5   10.240.0.4     <none>        Ubuntu 16.04.5 LTS   4.15.0-1035-azure   docker://3.0.1
k8s-master-44862260-0                Ready     master    3m        v1.11.5   10.255.255.5   <none>        Ubuntu 16.04.5 LTS   4.15.0-1035-azure   docker://3.0.1
$
$ aks-engine orchestrators --orchestrator kubernetes --version 1.11.5
{
  "orchestrators": [
    {
      "orchestratorType": "Kubernetes",
      "orchestratorVersion": "1.11.5",
      "upgrades": [
        {
          "orchestratorType": "",
          "orchestratorVersion": "1.11.6"
        },
        {
          "orchestratorType": "",
          "orchestratorVersion": "1.12.2"
        },
        {
          "orchestratorType": "",
          "orchestratorVersion": "1.12.4"
        }
      ]
    }
  ]
}
```

Pick a version, in our example I will use 1.12.2, and run the `aks-engine upgrade` command. You will need to pass the below arguments.

- Azure Subscription ID.
- AKS-Engine output directory.
- Azure region where the cluster is deployed.
- Resource Group.
- Kubernetes version.
- Authentication method. In our case `client_secret`.
- AAD Service Principal ID.
- AAD Service Principal Secret.

The output for the command should be similar to the one below. 

```
$ aks-engine upgrade --deployment-dir _output/k8s-lab-cl2/ --auth-method client_secret --location westeurope --client-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx --client-secret xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx --upgrade-version 1.12.2 --resource-group k8s-lab-cl2 --subscription-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx 
INFO[0000] validating...
INFO[0019] Name suffix: 44862260
INFO[0019] Gathering agent pool names...
INFO[0019] VM k8s-agentpool1-44862260-vmss_0 in VMSS k8s-agentpool1-44862260-vmss has a current version of Kubernetes:1.11.5 and a desired version of Kubernetes:1.12.2. Upgrading this node.
INFO[0019] VM k8s-agentpool1-44862260-vmss_1 in VMSS k8s-agentpool1-44862260-vmss has a current version of Kubernetes:1.11.5 and a desired version of Kubernetes:1.12.2. Upgrading this node.
INFO[0019] VM k8s-agentpool1-44862260-vmss_2 in VMSS k8s-agentpool1-44862260-vmss has a current version of Kubernetes:1.11.5 and a desired version of Kubernetes:1.12.2. Upgrading this node.
INFO[0019] Master VM name: k8s-master-44862260-0, orchestrator: Kubernetes:1.11.5 (MasterVMs)
INFO[0019] Upgrading to Kubernetes version 1.12.2
INFO[0019] Master nodes StorageProfile: ManagedDisks
INFO[0019] Prepping master nodes for upgrade...
INFO[0019] Resource count before running NormalizeResourcesForK8sMasterUpgrade: 11
INFO[0019] Evaluating if agent pool: master, resource: [concat(variables('masterVMNamePrefix'), copyIndex(variables('masterOffset')))] needs to be removed
INFO[0019] Evaluating if extension: [concat(variables('masterVMNamePrefix'), copyIndex(variables('masterOffset')),'/cse', '-master-', copyIndex(variables('masterOffset')))] needs to be removed
INFO[0019] Evaluating if extension: [concat(variables('masterVMNamePrefix'), copyIndex(variables('masterOffset')), '/computeAksLinuxBilling')] needs to be removed
INFO[0019] Resource count after running NormalizeResourcesForK8sMasterUpgrade: 11
INFO[0019] Total expected master count: 1
INFO[0019] Master nodes that need to be upgraded: 1
INFO[0019] Master nodes that have been upgraded: 0
INFO[0019] Starting upgrade of master nodes...
INFO[0019] masterNodesInCluster: 1
INFO[0019] Upgrading Master VM: k8s-master-44862260-0
INFO[0019] fetching VM: k8s-lab-cl2/k8s-master-44862260-0
INFO[0019] found nic name for VM (k8s-lab-cl2/k8s-master-44862260-0): k8s-master-44862260-nic-0
INFO[0019] deleting VM: k8s-lab-cl2/k8s-master-44862260-0
INFO[0019] waiting for vm deletion: k8s-lab-cl2/k8s-master-44862260-0
INFO[0110] deleting nic: k8s-lab-cl2/k8s-master-44862260-nic-0
INFO[0110] waiting for nic deletion: k8s-lab-cl2/k8s-master-44862260-nic-0
INFO[0120] deleting managed disk: k8s-lab-cl2/k8s-master-44862260-0_OsDisk_1_257d7386bcd74ba38d11578ad94c777a
INFO[0180] Master offset: 0
INFO[0180] Master pool set count to: 1 temporarily during upgrade...
INFO[0180] Starting ARM Deployment (master-19-02-07T18.31.02-556522995). This will take some time...
INFO[0496] Finished ARM Deployment (master-19-02-07T18.31.02-556522995). Succeeded
INFO[0501] Master VM: k8s-master-44862260-0 is ready
INFO[0501] Expected master count: 1, Creating 0 more master VMs
INFO[0501] Master VM: k8s-master-44862260-0 is ready
INFO[0502] Deploying the agent scale sets ARM template...
INFO[0502] Starting ARM Deployment (agentscaleset-19-02-07T18.36.23-1760152161). This will take some time...
INFO[0502] Master VM: k8s-master-44862260-0 is ready
INFO[0540] Finished ARM Deployment (agentscaleset-19-02-07T18.36.23-1760152161). Succeeded
INFO[0540] Upgrading VMSS k8s-agentpool1-44862260-vmss
INFO[0540] VMSS k8s-agentpool1-44862260-vmss current capacity is 3 and new capacity will be 4 while each node is swapped
INFO[0698] Successfully set capacity for VMSS k8s-agentpool1-44862260-vmss
INFO[0698] Draining node k8s-agentpool1-44862260-vmss000000
INFO[0698] Node k8s-agentpool1-44862260-vmss000000 has been marked unschedulable.
INFO[0698] 1 pods need to be removed/deleted
INFO[0698] kubernetes-dashboard-7b5859758b-f24jm pod successfully evicted
INFO[0698] Deleting VM k8s-agentpool1-44862260-vmss000000 in VMSS k8s-agentpool1-44862260-vmss
INFO[0759] Successfully deleted VM k8s-agentpool1-44862260-vmss000000 in VMSS k8s-agentpool1-44862260-vmss
INFO[0917] Successfully set capacity for VMSS k8s-agentpool1-44862260-vmss
INFO[0917] Draining node k8s-agentpool1-44862260-vmss000001
INFO[0917] Node k8s-agentpool1-44862260-vmss000001 has been marked unschedulable.
INFO[0917] 1 pods need to be removed/deleted
INFO[0917] kube-dns-8446b8bd4c-xr5rg pod successfully evicted
INFO[0917] Deleting VM k8s-agentpool1-44862260-vmss000001 in VMSS k8s-agentpool1-44862260-vmss
INFO[1098] Successfully deleted VM k8s-agentpool1-44862260-vmss000001 in VMSS k8s-agentpool1-44862260-vmss
INFO[1316] Successfully set capacity for VMSS k8s-agentpool1-44862260-vmss
INFO[1316] Draining node k8s-agentpool1-44862260-vmss000002
INFO[1316] Node k8s-agentpool1-44862260-vmss000002 has been marked unschedulable.
INFO[1316] 5 pods need to be removed/deleted
INFO[1316] heapster-855c6c8b5b-rtcx8 pod successfully evicted
INFO[1316] coredns-789f77df7-s5zkg pod successfully evicted
INFO[1316] kubernetes-dashboard-7b5859758b-sdjnq pod successfully evicted
INFO[1316] tiller-deploy-88c69b9b-gn6kg pod successfully evicted
INFO[1316] metrics-server-5fdc668b9b-sjrt6 pod successfully evicted
INFO[1316] Deleting VM k8s-agentpool1-44862260-vmss000002 in VMSS k8s-agentpool1-44862260-vmss
INFO[1557] Successfully deleted VM k8s-agentpool1-44862260-vmss000002 in VMSS k8s-agentpool1-44862260-vmss
INFO[1557] Completed upgrading VMSS k8s-agentpool1-44862260-vmss
INFO[1557] Completed upgrading all VMSS
INFO[1557] Cluster upgraded successfully to Kubernetes version 1.12.2
```

During the upgrade process `aks-engine` will go through the different instances that form the cluster, starting with the masters, and cordon the node, delete the instance, provision a new instance, install the new Kubernetes version and finally add the new node to the cluster. 

Verify the new Kubernetes version after the upgrade is finished. 

```
$ kubectl get nodes -o wide
NAME                                 STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k8s-agentpool1-44862260-vmss000003   Ready    agent    49m   v1.12.2   10.240.0.127   <none>        Ubuntu 16.04.5 LTS   4.15.0-1036-azure   docker://3.0.1
k8s-agentpool1-44862260-vmss000004   Ready    agent    45m   v1.12.2   10.240.0.4     <none>        Ubuntu 16.04.5 LTS   4.15.0-1036-azure   docker://3.0.1
k8s-agentpool1-44862260-vmss000005   Ready    agent    39m   v1.12.2   10.240.0.35    <none>        Ubuntu 16.04.5 LTS   4.15.0-1036-azure   docker://3.0.1
k8s-master-44862260-0                Ready    master   11h   v1.12.2   10.255.255.5   <none>        Ubuntu 16.04.5 LTS   4.15.0-1036-azure   docker://3.0.1
```

As you can see the master and the nodes are in Kubernetes 1.12.2 version, also since the availability profile of my nodes is `VirtualMachineScaleSets` the nodes are newly deployed. 

--Juanma