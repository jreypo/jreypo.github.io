---
title: AKS, the new Kubernetes managed service on Azure
date: 2017-10-24
tags:
- aks
- azure
- cloud-native
- containers
- docker
- kubernetes
showComments: true
---

AKS is a new Azure service that provides customers with the possibility of deploying a managed Kubernetes cluster on their Azure subscription. Similarly to Google GKE the new service will allow access to the nodes only and the masters will be managed by Microsoft. Read the [official announcement here](https://azure.microsoft.com/en-us/blog/introducing-azure-container-service-aks-managed-kubernetes-and-azure-container-registry-geo-replication/), includes some nice videos by [Scott Hanselman](https://twitter.com/shanselman) and [Gabe Monroy](https://twitter.com/gabrtv).

Microsoft Azure documentation has been also updated with the new service, check it out on this [link](https://docs.microsoft.com/en-us/azure/aks/).

Finally a new [GitHub repo](https://github.com/Azure/AKS) has been created for users to report bugs and for service annoucements and updates from the AKS team.

Let's deploy our first AKS cluster. We can use the Cloud Shell on the Azure portal or Azure CLI, I will use the last one from WSL on my Windwos 10 laptop. Before being able to do it we will need to upgrade the cli to its newest version, in my case I did it in all my systems with `sudo apt-get update && sudo apt-get upgrade azure-cli` on Ubuntu 16.04 on WSL and `dnf update -y azure-cli` on my Fedora 26 workstation at home.

After the upgrade the new `aks` option will appear.

```text
$ az aks -h

Group
    az aks: Manage Kubernetes clusters.

Commands:
    browse         : Open a web browser to the dashboard for a managed Kubernetes cluster.
    create         : Create a new Azure managed Kubernetes cluster.
    delete         : Delete a managed Kubernetes cluster.
    get-credentials: Get credentials to access a managed Kubernetes cluster.
    get-versions   : Get versions available to upgrade a managed Kubernetes cluster.
    install-cli    : Install kubectl, a command-line interface for Kubernetes clusters.
    list           : List managed Kubernetes clusters.
    scale          : Scale the agent pool in a managed Kubernetes cluster.
    show           : Show a managed Kubernetes cluster.
    upgrade        : Upgrade a managed Kubernetes cluster to a newer version.
    wait           : Wait for a managed Kubernetes cluster to reach a desired state.
```

If you are using Azure CLI from your system instead of the Cloud Shell it is very important to re-register the `ContainerService` provider in order to have access to the AKS resource type.

```text
$ az provider register --namespace Microsoft.ContainerService
Registering is still on-going. You can monitor using 'az provider show -n Microsoft.ContainerService'
$ az provider show -n Microsoft.ContainerService
Namespace                   RegistrationState
--------------------------  -------------------
Microsoft.ContainerService  Registered
```

Create a new resource group and a new service principal or reuse an existing one, like with standard ACS. For now, AKS is only available in UK West and West US 2 Azure regions so remember to set the location of the resource group to one of those two regions. Like on ACS we can specify the number of agents and the size of them and as a new addition we can set the Kubernetes version.

```text
$ az group create -n aksrg -l westus2
Location    Name
----------  ------
westus2     aksrg
$ az aks create -g aksrg -n aks-cl1 -p aks-cl1 -k 1.7.7 -c 3 --service-principal <REDACTED> --client-secret <REDACTED> -u azuser --verbose
Use existing SSH public key file: /home/jurey/.ssh/id_rsa.pub
Location    Name     ResourceGroup
----------  -------  ---------------
westus2     aks-cl1  aksrg
```

Using Azure CLI retrieve the credentials and take a look at the cluster.

```text
$ az aks get-credentials -g aksrg -n aks-cl1
Merged "aks-cl1" as current context in /home/jurey/.kube/config
$ kubectl get node
NAME                        STATUS    AGE       VERSION
aks-agentpool1-74041364-0   Ready     11m       v1.7.7
aks-agentpool1-74041364-1   Ready     11m       v1.7.7
aks-agentpool1-74041364-2   Ready     11m       v1.7.7
```

Now let's upgrade those nodes to a newer version :D

```text
$ az aks get-versions -g aksrg -n aks-cl1
Name     ResourceGroup    MasterVersion    MasterUpgrades    AgentPoolVersion    AgentPoolUpgrades
-------  ---------------  ---------------  ----------------  ------------------  -------------------
default  aksrg            1.7.7            1.8.1             1.7.7               1.8.1
$ az aks upgrade -g aksrg -n aks-cl1 -k 1.8.1 --no-wait
Kubernetes may be unavailable during cluster upgrades.
Are you sure you want to perform this operation? (y/n): y
```

This command will upgrade first the masters as you can expect, during that time the cluster will be unavailable. After a few minutes the masters will be back, and the nodes will be gradually upgraded.

```text
$ az aks get-versions -g aksrg -n aks-cl1
Name     ResourceGroup    MasterVersion    MasterUpgrades    AgentPoolVersion    AgentPoolUpgrades
-------  ---------------  ---------------  ----------------  ------------------  -------------------
default  aksrg            1.8.1            None available    1.8.1               None available
$ kubectl get node -o wide
Unable to connect to the server: net/http: TLS handshake timeout
$ kubectl get node -o wide
NAME                        STATUS    AGE       VERSION   EXTERNAL-IP   OS-IMAGE                      KERNEL-VERSION
aks-agentpool1-74041364-0   Ready     1m        v1.8.1    <none>        Debian GNU/Linux 8 (jessie)   4.11.0-1013-azure
aks-agentpool1-74041364-1   Ready     26m       v1.7.7    <none>        Debian GNU/Linux 8 (jessie)   4.11.0-1013-azure
aks-agentpool1-74041364-2   Ready     27m       v1.7.7    <none>        Debian GNU/Linux 8 (jessie)   4.11.0-1013-azure
```

And after some time all the nodes will appear with the new version.

```text
$ kubectl get node -o wide
NAME                        STATUS    AGE       VERSION   EXTERNAL-IP   OS-IMAGE                      KERNEL-VERSION
aks-agentpool1-74041364-0   Ready     19m       v1.8.1    <none>        Debian GNU/Linux 8 (jessie)   4.11.0-1013-azure
aks-agentpool1-74041364-1   Ready     11m       v1.8.1    <none>        Debian GNU/Linux 8 (jessie)   4.11.0-1013-azure
aks-agentpool1-74041364-2   Ready     3m        v1.8.1    <none>        Debian GNU/Linux 8 (jessie)   4.11.0-1013-azure
```

When the upgrade is done we can add mode nodes to try the scale feature, very similar to the ACS scaling feature.

```text
az aks scale --resource-group aksrg --name aks-cl1 --agent-count 5 --no-wait --verbose
```

Watch the progress with `kubectl` until the new nodes are added the cluster.

```text
$ kubectl get node -w
NAME                        STATUS    AGE       VERSION
aks-agentpool1-74041364-0   Ready     28m       v1.8.1
aks-agentpool1-74041364-1   Ready     20m       v1.8.1
aks-agentpool1-74041364-2   Ready     11m       v1.8.1
aks-agentpool1-74041364-3   Ready     1m        v1.8.1
aks-agentpool1-74041364-4   Ready     49s       v1.8.1
```

This new service opens new possibilities and improves the offering on Azure around containers, which in the end is good for Microsoft customers. I encourage you to try it and left me any comment below with your findings, tips and impressions. Of course if you find any issues please open a bug on the above mentioned GitHub repository.

--Juanma
