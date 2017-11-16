---
layout: post
title: Howto enable RBAC on you existing ACS deployed Kubernetes cluster
date: 2017-11-16
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
- Azure
- Containers
- Kubernetes
- ACS
- Azure Container Service
- Cloud-Native
- Azure Active Directory
author: juan_manuel_rey
comments: true
---

On a my [post about AAD integration with Kubernetes 1.8]({% post_url 2017-11-08-kubernetes-18-with-rbac-enabled-and-azure-active-directory-integration %}) I explained how to deploy a Kubernetes 1.8 cluster with RBAC enabled using [ACS-Engine](github.com/Azure/acs-engine). However after that post I got a couple of questions about how to enable RBAC on a cluster deployed using Azure Container Service. 

Currently in ACS the Kubernetes version is 1.7.7.

```
$ kubectl get nodes -o wide
NAME                    STATUS    AGE       VERSION   EXTERNAL-IP   OS-IMAGE                      KERNEL-VERSION
k8s-agent-537d3417-0    Ready     1d        v1.7.7    <none>        Debian GNU/Linux 8 (jessie)   4.4.0-98-generic
k8s-agent-537d3417-1    Ready     1d        v1.7.7    <none>        Debian GNU/Linux 8 (jessie)   4.4.0-98-generic
k8s-agent-537d3417-2    Ready     1d        v1.7.7    <none>        Debian GNU/Linux 8 (jessie)   4.4.0-98-generic
k8s-master-537d3417-0   Ready     1d        v1.7.7    <none>        Debian GNU/Linux 8 (jessie)   4.4.0-98-generic
```

The process to enable RBAC on this cluster is pretty straigh forward. 

1. SSH into the master node(s). 
2. Edit `/etc/kubernetes/manifests/kube-apiserver.yaml` and locate the `command` property. Should be something like this.
```yaml
command:
    - "/hyperkube"
    - "apiserver"
    - "--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota"
    - "--address=0.0.0.0"
    - "--allow-privileged"
    - "--insecure-port=8080"
    - "--secure-port=443"
    - "--cloud-provider=azure"
    - "--cloud-config=/etc/kubernetes/azure.json"
    - "--service-cluster-ip-range=10.0.0.0/16"
    - "--etcd-servers=http://127.0.0.1:2379"
    - "--etcd-quorum-read=true"
    - "--advertise-address=10.240.255.15"
    - "--tls-cert-file=/etc/kubernetes/certs/apiserver.crt"
    - "--tls-private-key-file=/etc/kubernetes/certs/apiserver.key"
    - "--client-ca-file=/etc/kubernetes/certs/ca.crt"
    - "--service-account-key-file=/etc/kubernetes/certs/apiserver.key"
    - "--storage-backend=etcd2"
    - "--v=4"
```
3. Add `--authorization-mode=RBAC`.
4. Reboot the nodes.

And we are done. After the reboot follow the same instructions detailed in my post about RBAC with AAD on Kubernetes 1.8 to integrate it with your Azure Active Directory Domain. Also remember that ACS deploys Kuberneetes 1.7 and RBAC was still experimental on that version and may behave different from 1.8.

--Juanma. 