---
layout: post
title: Scaling operations with AKS-Engine
date: 2019-06-13 05:12:00 +0100
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

After reviewing how to perform Kubernetes version upgrades with AKS-Engine in a [previous post]({% post_url 2019-02-07-kubernetes-version-upgrade-with-aks-engine %}) the next logical step is show how to scale our Kubernetes clusters with AKS-Engine. I will cover manual scaling of the cluster, of course you can always deploy and configure the [Kubernetes Cluster Autoscaler](https://github.com/kubernetes/autoscaler). 

There are several scaling scenarios that can achieved using AKS-Engine:

- Resize an existing nodepool.
- Add a new nodepool.
- Remove a nodepool.

Keep in mind also that the scaling operations will require the API model file used originally to deploy the cluster.

# Resize an existing nodepool

To resize an existing nodepool the best way is tu use the `aks-engine scale` command. The arguments are very similar to the ones used for the upgrade and include:

- `--deployment-dir` The location of the output files creted during the generate operation.
- `--auth-method` The authentication method, can be `client_secret` or `client_certificate`, in our example we are using the first one.
- `--client-id` AAD Service Principal ID.
- `--client-secret` AAD Service Principal Secret.
- `--subscription-id` Azure Subscription ID.
- `--location` Location, the Azure region where the cluster is deployed on.
- `--master-FQDN` FQDN of the master.
- `--resource-group` Resource group.
- `--node-pool` Nodepool name.
- `--new-node-count` New number of nodes.

```
$ aks-engine scale --location westeurope --subscription-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx --resource-group k8s-lab-cl2 --node-pool agentpool1 --master-FQDN https://my-cluster.westeurope.cloudapp.azure.com --new-node-count 4 --auth-method client_secret --client-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx --client-secret xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx --deployment-dir ./_output/k8s-lab-cl2/
INFO[0000] validating...
INFO[0014] Name suffix: 44862260
INFO[0014] Found no resources with type Microsoft.Network/routeTables in the template.  source="scaling command line"
INFO[0014] Starting ARM Deployment (k8s-lab-cl2-1860437642). This will take some time...
INFO[0289] Finished ARM Deployment (k8s-lab-cl2-1860437642). Succeeded
```

After the command is completed succesfully check that a new node has been added to the cluster. 

```
$ kubectl get nodes
NAME                                 STATUS   ROLES    AGE    VERSION
k8s-agentpool1-44862260-vmss000003   Ready    agent    60d    v1.12.2
k8s-agentpool1-44862260-vmss000004   Ready    agent    60d    v1.12.2
k8s-agentpool1-44862260-vmss000005   Ready    agent    60d    v1.12.2
k8s-agentpool1-44862260-vmss000006   Ready    agent    3m8s   v1.12.2
k8s-master-44862260-0                Ready    master   61d    v1.12.2
```

# Add a new nodepool

To add a new `nodepool` to your cluster you will need to edit the `apimodel.json` file, in the `_output/<cluster-fqdn>` directory, and add a `nodepool` entry in the `agentPoolProfiles ` array. For example my current `agentPoolProfiles` looks like this one:

```json
"agentPoolProfiles": [
    {
    "name": "agentpool1",
    "count": 4,
    "vmSize": "Standard_B2s",
    "osType": "Linux",
    "availabilityProfile": "VirtualMachineScaleSets",
    "storageProfile": "ManagedDisks",
    "distro": "aks",
    "kubernetesConfig": {
        "kubeletConfig": {
        "--address": "0.0.0.0",
        "--allow-privileged": "true",
        "--anonymous-auth": "false",
        "--authorization-mode": "Webhook",
        "--azure-container-registry-config": "/etc/kubernetes/azure.json",
        "--cgroups-per-qos": "true",
        "--client-ca-file": "/etc/kubernetes/certs/ca.crt",
        "--cloud-config": "/etc/kubernetes/azure.json",
        "--cloud-provider": "azure",
        "--cluster-dns": "10.0.0.10",
        "--cluster-domain": "cluster.local",
        "--enforce-node-allocatable": "pods",
        "--event-qps": "0",
        "--eviction-hard": "memory.available<100Mi,nodefs.available<10%,nodefs.inodesFree<5%",
        "--feature-gates": "PodPriority=true",
        "--image-gc-high-threshold": "85",
        "--image-gc-low-threshold": "80",
        "--image-pull-progress-deadline": "30m",
        "--keep-terminated-pod-volumes": "false",
        "--kubeconfig": "/var/lib/kubelet/kubeconfig",
        "--max-pods": "30",
        "--network-plugin": "cni",
        "--node-status-update-frequency": "10s",
        "--non-masquerade-cidr": "0.0.0.0/0",
        "--pod-infra-container-image": "k8s.gcr.io/pause-amd64:3.1",
        "--pod-manifest-path": "/etc/kubernetes/manifests",
        "--pod-max-pids": "100"
        }
    },
    "acceleratedNetworkingEnabled": false,
    "acceleratedNetworkingEnabledWindows": false,
    "fqdn": "",
    "preProvisionExtension": null,
    "extensions": [],
    "singlePlacementGroup": true
    }
],
```

I will need to copy the `nodepool1` entry and modify it accordingly.

```json
{
"name": "agentpool2",
"count": 2,
"vmSize": "Standard_DS4_v2s",
"osType": "Linux",
"availabilityProfile": "VirtualMachineScaleSets",
"storageProfile": "ManagedDisks",
"distro": "aks",
"kubernetesConfig": {
    "kubeletConfig": {
    "--address": "0.0.0.0",
    "--allow-privileged": "true",
    "--anonymous-auth": "false",
    "--authorization-mode": "Webhook",
    "--azure-container-registry-config": "/etc/kubernetes/azure.json",
    "--cgroups-per-qos": "true",
    "--client-ca-file": "/etc/kubernetes/certs/ca.crt",
    "--cloud-config": "/etc/kubernetes/azure.json",
    "--cloud-provider": "azure",
    "--cluster-dns": "10.0.0.10",
    "--cluster-domain": "cluster.local",
    "--enforce-node-allocatable": "pods",
    "--event-qps": "0",
    "--eviction-hard": "memory.available<100Mi,nodefs.available<10%,nodefs.inodesFree<5%",
    "--feature-gates": "PodPriority=true",
    "--image-gc-high-threshold": "85",
    "--image-gc-low-threshold": "80",
    "--image-pull-progress-deadline": "30m",
    "--keep-terminated-pod-volumes": "false",
    "--kubeconfig": "/var/lib/kubelet/kubeconfig",
    "--max-pods": "110",
    "--network-plugin": "cni",
    "--node-status-update-frequency": "10s",
    "--non-masquerade-cidr": "0.0.0.0/0",
    "--pod-infra-container-image": "k8s.gcr.io/pause-amd64:3.1",
    "--pod-manifest-path": "/etc/kubernetes/manifests",
    "--pod-max-pids": "100"
    }
},
```

Once the `apimodel` file is modified run `aks-engine generate --api-model _output/<clustername>/apimodel.json`. This operation will update the original `azuredeploy.json` and `azuredeploy.parameters.json` files used durign the ARM template deployment. 

After the `aks-engine generate` operation is done then run  `az group deployment create --template-file _output/<clustername>/azuredeploy.json --parameters _output/<clustername>/azuredeploy.parameters.json --resource-group` <my-resource-group>`

# Remove a nodepool

Removing a nodepool from an existing cluster is very similar to the adding operation, just edit the `_output/<clustername>/apimodel.json` file, remove the nodepool entry and then run again the `aks-engine generate` and `az group deployment create` commands.

However there is a catch, you have to manually drain the nodes in your nodepool before executing `az group deployment create`. After the operation is finished review your resource group to verify that every related resource has been correctly eliminated.

Hope the post helps to clarify the different scaling scenarios with AKS-Engine. Comments as always are welcome. 

--Juanma