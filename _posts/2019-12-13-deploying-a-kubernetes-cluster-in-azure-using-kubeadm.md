---
title: Deploying a Kubernetes cluster in Azure using kubeadm
date: 2019-12-13 00:15:00 +0100
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
- DevOps
tags:
- Microsoft
- Containers
- Kubernetes
- Cloud-Native
- AKS
- Kubeadm
- DevOps
- Azure
author: juan_manuel_rey
comments: true
---

The easiest way to have a [Kubernetes](https://kubernetes.io/) cluster up and running in Azure in a short amount of time is by using [AKS service](https://azure.microsoft.com/es-es/services/kubernetes-service/), also if you want a more granular control of your cluster or a more customized cluster you can alway use [AKS-Engine](https://github.com/azure/aks-engine).

However this time I wanted to take a different approach and use a more widely used tool very popular amongst the Kubernetes community like `kuebadm`. I like `kubeadm` as a fantastic way to learn the internals of Kubernetes, also I used the content of this post as part of the preparation for [CKA certification](https://www.cncf.io/certification/cka/) exam, which I am planning to take in December or January.

## Create Azure infrastructure

The first thing we myst do is create the necessary infrastructure in our subscription, this includes the instances and the network.

Create the resource group.

```
az group create --name k8s-lab-rg3 --location westeurope
```

Create a VNet, during the creation of the VNet we will declare as well the subnet for our cluster. 

```
az network vnet create --name k8s-lab-vnet --resource-group k8s-lab-rg3 --location westeurope --address-prefixes 172.10.0.0/16 --subnet-name k8s-lab-net1 --subnet-prefixes 172.10.1.0/24
```

Create the instances, one master and three nodes. For my lab I am using the laster `UbuntuLTS` image and `Standard_DS2_v2` for the size of the instances.

```bash
#!/bin/bash

RG=k8s-lab-rg3
LOCATION=westeurope
SUBNET=$(az network vnet show --name k8s-lab-vnet -g $RG --query subnets[0].id -o tsv)

# Master instance
echo "Creating Kubernetes Master"
az vm create --name kube-master \
   --resource-group $RG \
   --location $LOCATION \
   --image UbuntuLTS \
   --admin-user azureuser \
   --ssh-key-values ~/.ssh/id_rsa.pub \
   --size Standard_DS2_v2 \
   --data-disk-sizes-gb 10 \
   --subnet $SUBNET \
   --public-ip-address-dns-name kube-master-lab

# Nodes intances

az vm availability-set create --name kubeadm-nodes-as --resource-group $RG

for i in 0 1 2; do 
    echo "Creating Kubernetes Node ${i}"
    az vm create --name kube-node-${i} \
       --resource-group $RG \
       --location $LOCATION \
       --availability-set kubeadm-nodes-as \
       --image UbuntuLTS \
       --admin-user azureuser \
       --ssh-key-values ~/.ssh/id_rsa.pub \
       --size Standard_DS2_v2 \
       --data-disk-sizes-gb 10 \
       --subnet $SUBNET \
       --public-ip-address-dns-name kube-node-lab-${i}
done

az vm list --resource-group $RG -d
```

## Prepare the cluster master and node instances 

With the instances up and running we need to install the software we will use to create our Kubernetes cluster.

Access the master and install `docker`, remember to install a Docker release validated for Kubernetes. In my case I will use Kubernetes 1.16 and Docker 18.09.

```
azureuser@kube-master-lab:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK
azureuser@kube-master-lab:~$ sudo apt-key fingerprint 0EBFCD88
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]

azureuser@kube-master-lab:~$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
azureuser@kube-master-lab:~$ sudo apt-get update 
...
azureuser@kube-master-lab:~$ sudo apt-get install -y docker-ce=5:18.09.9~3-0~ubuntu-bionic docker-ce-cli containerd.io
```

Configure Docker daemon for Kubernetes.

```
azureuser@kube-master-lab:~$ cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
azureuser@kube-master-lab:~$ sudo mkdir -p /etc/systemd/system/docker.service.d
azureuser@kube-master-lab:~$ sudo systemctl daemon-reload
azureuser@kube-master-lab:~$ sudo systemctl restart docker
azureuser@kube-master-lab:~$

```

Configure Kubernetes `apt` repo and install `kubeadm`.

```
azureuser@kube-master-lab:~$ sudo apt-get install -y apt-transport-https
...
azureuser@kube-master-lab:~$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
OK
azureuser@kube-master-lab:~$ cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
azureuser@kube-master-lab:~$ sudo apt-get update
...
azureuser@kube-master-lab:~$ sudo apt-get install -y kubelet kubeadm kubectl
...
azureuser@kube-master-lab:~$ sudo apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
azureuser@kube-master-lab:~$
```

Repeat the same process for each of the nodes either manually or by using the below script, which can also be found as a [Gist on my GitHub](https://gist.github.com/jreypo/8264157231a649fe4d65762917d6a27f).

```bash
#!/bin/bash

echo "Installing Docker..."

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update && sudo apt-get install -y docker-ce=5:18.09.9~3-0~ubuntu-bionic docker-ce-cli containerd.io

echo "Configuring Docker..."

sudo cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo mkdir -p /etc/systemd/system/docker.service.d
sudo systemctl daemon-reload
sudo systemctl restart docker

echo "Installing Kubernetes components..."

sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add 
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Create the cluster

### Create `kubeadm` configuration

To bootstrap a cluster integrated with Azure, this is using the Azure cloud provider, using `kubeadm` we will need a `kuebadm` configuration file. In this file we will specify the Cluster Manager and API Server parameters instructing `kubeadm` to configure it with `--cloud-provider=azure` flag. For more information on Kubernetes cloud providers with `kubeadm` review the official [Kubernetes documentation](https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/?source=post_page-----357210e2eb50----------------------#kubeadm).

Below is my `kubeadm.yaml` configuration file, you can use it and adjust the networking parameters to your preference.

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
nodeRegistration:
  kubeletExtraArgs:
    cloud-provider: "azure"
    cloud-config: "/etc/kubernetes/cloud.conf"
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.13.0
apiServer:
  extraArgs:
    cloud-provider: "azure"
    cloud-config: "/etc/kubernetes/cloud.conf"
  extraVolumes:
  - name: cloud
    hostPath: "/etc/kubernetes/cloud.conf"
    mountPath: "/etc/kubernetes/cloud.conf"
controllerManager:
  extraArgs:
    cloud-provider: "azure"
    cloud-config: "/etc/kubernetes/cloud.conf"
  extraVolumes:
  - name: cloud
    hostPath: "/etc/kubernetes/cloud.conf"
    mountPath: "/etc/kubernetes/cloud.conf"
networking:
  serviceSubnet: "10.12.0.0/16"
  podSubnet: "10.11.0.0/16"
```

Next create `/etc/kubernetes/cloud.conf` file, it will contain the configuration for the Azure Cloud Provider.

```json
{
    "cloud":"AzurePublicCloud",
    "tenantId": "xxxx",
    "subscriptionId": "xxxx",
    "aadClientId": "xxxx",
    "aadClientSecret": "xxxx",
    "resourceGroup": "k8s-lab-rg4",
    "location": "westeurope",
    "vmType": "standard",
    "subnetName": "k8s-lab-net1",
    "securityGroupName": "kube-masterNSG",
    "vnetName": "k8s-lab-vnet",
    "vnetResourceGroup": "",
    "routeTableName": "",
    "primaryAvailabilitySetName": "kubeadm-nodes-as",
    "primaryScaleSetName": "",
    "cloudProviderBackoffMode": "v2",
    "cloudProviderBackoff": true,
    "cloudProviderBackoffRetries": 6,
    "cloudProviderBackoffDuration": 5,
    "cloudProviderRatelimit": true,
    "cloudProviderRateLimitQPS": 10,
    "cloudProviderRateLimitBucket": 100,
    "cloudProviderRatelimitQPSWrite": 10,
    "cloudProviderRatelimitBucketWrite": 100,
    "useManagedIdentityExtension": false,
    "userAssignedIdentityID": "",
    "useInstanceMetadata": true,
    "loadBalancerSku": "Basic",
    "disableOutboundSNAT": false,
    "excludeMasterFromStandardLB": false,
    "providerVaultName": "",
    "maximumLoadBalancerRuleCount": 250,
    "providerKeyName": "k8s",
    "providerKeyVersion": ""
}
```

### Bootstrap the master node

Initialize the master, or control plane node, passing `kuebadm.yaml` as configuration parameter. Make sure that the instance name in Azure is the same as the hostname or `kubeadm` will fail fail to initialize the `kubelet`.

```
azureuser@kube-master-lab:~$ sudo kubeadm init --config kubeadm.yml
[init] Using Kubernetes version: v1.16.0
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
...
...
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.10.1.4:6443 --token 81l08m.0g09hbdekfxczgs0 \
    --discovery-token-ca-cert-hash sha256:a868e59818db186a2cb03a32c2478d7abafbf4ceae471532e1152fb4949298fd
azureuser@kube-master-lab:~$
```

As the output suggests create a `kubeconfig` file to start using the cluster.

```
azureuser@kube-master-lab:~$ mkdir -p $HOME/.kube
azureuser@kube-master-lab:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
azureuser@kube-master-lab:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
azureuser@kube-master-lab:~$ kubectl get nodes -o wide
NAME          STATUS     ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP     OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
kube-master   NotReady   master   12m   v1.16.3   172.10.1.4    51.144.178.87   Ubuntu 18.04.3 LTS   5.0.0-1025-azure   docker://18.9.9
azureuser@kube-master:~$
```

### Install a networking add-on

Next is to install a networking add-on in the master, in my example I am going to use [Calico](https://www.projectcalico.org/) but can you choose another if you want. First retrieve `calico.yaml` manifest from `https://docs.projectcalico.org/v3.8/manifests/calico.yaml` edit it and replace `CALICO_IPV4POOL_CIDR` value of `192.168.0.0/16` with the one from `podSubnet` property defined in ourt `kubeadm.yaml` file, in my case `10.11.0.0/16`.

```yaml
- name: CALICO_IPV4POOL_CIDR
value: "10.11.0.0/16"
```

Then apply the manifest with `kubectl`.

```
azureuser@kube-master-lab:~$ kubectl apply -f calico.yaml
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
azureuser@kube-master-lab:~$
azureuser@kube-master-lab:~$ kubectl get pods -n kube-system
NAME                                      READY   STATUS    RESTARTS   AGE
calico-kube-controllers-55754f75c-b42wk   1/1     Running   0          108s
calico-node-gmv55                         1/1     Running   0          109s
coredns-5644d7b6d9-2l49m                  1/1     Running   0          9m12s
coredns-5644d7b6d9-hhqq2                  1/1     Running   0          9m12s
etcd-kube-master                          1/1     Running   0          8m12s
kube-apiserver-kube-master                1/1     Running   0          8m16s
kube-controller-manager-kube-master       1/1     Running   0          8m11s
kube-proxy-r4pfn                          1/1     Running   0          9m12s
kube-scheduler-kube-master                1/1     Running   0          8m23s
azureuser@kube-master-lab:~$ kubectl get nodes -o wide
NAME          STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP     OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
kube-master   Ready    master   11m   v1.16.3   172.10.1.4    51.144.178.87   Ubuntu 18.04.3 LTS   5.0.0-1025-azure   docker://18.9.9
azureuser@kube-master=lab:~$
```

### Bootstrap the nodes

SSH into the first node and execute `kubeadm join` from master `kuebadm init` output. If you did not take note of the command or more than 24 hours have passed do not worry since we can easily reconstruct it with the following commands.

```
azureuser@kube-master-lab:~$ kubeadm token create
bvbjmy.z2m3y0mu9gtvar3s
azureuser@kube-master-lab:~$ kubeadm token list
TOKEN                     TTL       EXPIRES                USAGES                   DESCRIPTION   EXTRA GROUPS
bvbjmy.z2m3y0mu9gtvar3s   23h       2019-12-12T10:15:21Z   authentication,signing   <none>        system:bootstrappers:kubeadm:default-node-token
azureuser@kube-master-lab:~$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
>    openssl dgst -sha256 -hex | sed 's/^.* //'
b5b9429546c8cdf4accf006250558551240a56371528f8bff1a85e401fea4be2
azureuser@kube-master-lab:~$
```

Now login back into your first node and execute the `kubeadm join` command using the token as argument for the `--token` option and the hash key with format `sha256:<your_hasch>` as argument for the `--discovery-token-ca-cert-hash` option.

```
azureuser@kube-node-0:~$ sudo kubeadm join 172.10.1.4:6443 --token bvbjmy.z2m3y0mu9gtvar3s --discovery-token-ca-cert-hash sha256:b5b9429546c8cdf4accf006250558551240a56371528f8bff1a85e401fea4be2
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.16" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

azureuser@kube-node-0:~$
```

Repeat the process for each node, you can verify the status of the nodes from the master with `kubectl get nodes`.

```
azureuser@kube-master-lab:~$ kubectl get nodes -o wide
NAME          STATUS     ROLES    AGE    VERSION   INTERNAL-IP   EXTERNAL-IP     OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
kube-master   Ready      master   8d     v1.16.3   172.10.1.4    51.144.178.87   Ubuntu 18.04.3 LTS   5.0.0-1025-azure   docker://18.9.9
kube-node-0   Ready      <none>   8m7s   v1.16.3   172.10.1.5    <none>          Ubuntu 18.04.3 LTS   5.0.0-1025-azure   docker://18.9.9
kube-node-1   Ready      <none>   91s    v1.16.3   172.10.1.6    <none>          Ubuntu 18.04.3 LTS   5.0.0-1025-azure   docker://18.9.9
kube-node-2   NotReady   <none>   19s    v1.16.3   172.10.1.7    <none>          Ubuntu 18.04.3 LTS   5.0.0-1025-azure   docker://18.9.9
azureuser@kube-master-lab:~$
```

With the nodes properly joined the cluster is ready to be used.

Thanks for reading! Hope this whole procedure has been helpful and instructive, as always if you have any comments, questions or suggestions please leave them in the comments or reach out to me on Twitter.

--Juanma
