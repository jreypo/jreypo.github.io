---
title: My quick take on Helm
date: 2017-09-06
tags:
- azure
- cloud-native
- containers
- docker
- kubernetes
showComments: true
---

[**Helm**](https://helm.sh/) is one of those tools I've been watching from a safe distance for some time, always curious but never find the time to play with it. I decided to change that, specially since Helm was created by the awesome folks at DEIS which are now part of Microsoft.

## What is Helm

When DEIS introduced Helm for the first time it was described as The Kubernetes Package Manager, an open source project that allows to package, install and manage Kubernetes applications. During my experience these past weeks that is what I found, if you are familiar with Linux package managers like `apt` or `dnf` then you will feel at home using `helm` client to deploy charts on your Kubernetes clusters.

Helms is made of two differentiated components, `helm` command line client and Tiller which is the server-side component that will be installed on the cluster and that will interact with Kubernetes API server. [Helm documentation](https://docs.helm.sh/) provides much more details around the architecture of Helm.

## Install and configure Helm

Helm is currently available for Linux and macOS, there is an experimental version for Windows but who needs a Window version when you have Windows Subsystem for Linux. Yes Helm can be installed on a Linux distro running on WSL using the Linux installation script, actually I installed Helm on Ubuntu 16.04, which runs on my Windows 10 laptop on top of Windows Subsystem for Linux.

```text
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.s
$ chmod 700 get_helm.sh
$ ./get_helm.sh
Downloading https://kubernetes-helm.storage.googleapis.com/helm-v2.6.1-linux-amd64.tar.gz
Preparing to install into /usr/local/bin
[sudo] password for jurey:
helm installed into /usr/local/bin/helm
Run 'helm init' to configure helm.
```

Executing `helm init` will initialize Helm locally, connect to the default Kubernetes cluster and deploy Tiller under the `kube-system` namespace. If your Kubernetes cluster has been deployed using Azure Container Service then tiller comes already deployed, execute `helm init --upgrade` instead.

```text
$ helm init --upgrade
Creating /home/jurey/.helm
Creating /home/jurey/.helm/repository
Creating /home/jurey/.helm/repository/cache
Creating /home/jurey/.helm/repository/local
Creating /home/jurey/.helm/plugins
Creating /home/jurey/.helm/starters
Creating /home/jurey/.helm/cache/archive
Creating /home/jurey/.helm/repository/repositories.yaml
$HELM_HOME has been configured at /home/jurey/.helm.

Tiller (the Helm server-side component) has been upgraded to the current version.
Happy Helming!
```

However right now there is bug with Tiller on ACS, a few seconds after you upgrade Tiller a new pod with the previous version (2.5.1) will be re-deployed and the upgraded version will be removed. This bug has been reported on [ACS Github repository](https://github.com/Azure/ACS/issues/55), Tiller addon has been marked as a cluster service therefore Kubernetes will not allow any updates to it through the API which what `helm` does, instead the only way to update the version is by modifying the YAML manifest defining the deployment on the master(s). ACS team has already fix the issue and will be rolled up through all Azure regions during the next weeks.

In the meantime the quick fix is to update the deployment manifest located in `/etc/kubernetes/addons/kube-tiller-deployment.yaml` and replace `image: gcrio.azureedge.net/kubernetes-helm/tiller:v2.5.1` with the correct version.

After Helm initialization list the pods running under the `kube-system` namespace and you will see a Tiller one.

```text
$ kubectl get pod --namespace kube-system
NAME                                            READY     STATUS    RESTARTS   AGE
heapster-2708163903-9lg8w                       2/2       Running   0          41s
kube-addon-manager-k8s-master-c188e3da-0        1/1       Running   0          7d
kube-apiserver-k8s-master-c188e3da-0            1/1       Running   0          7d
kube-controller-manager-k8s-master-c188e3da-0   1/1       Running   0          7d
kube-dns-v20-pkqsp                              3/3       Running   0          7d
kube-dns-v20-zj2jg                              3/3       Running   0          7d
kube-proxy-qr3l2                                1/1       Running   0          7d
kube-proxy-sr40f                                1/1       Running   0          7d
kube-proxy-wr0kz                                1/1       Running   0          7d
kube-proxy-wtsbx                                1/1       Running   0          7d
kube-scheduler-k8s-master-c188e3da-0            1/1       Running   0          7d
kubernetes-dashboard-3995387264-npwk5           1/1       Running   0          7d
tiller-deploy-3019006398-k2tl2                  1/1       Running   0          43s
```

## Deploy a chart

I found the process to deploy a chart very intuitive, like a said is very much like using any other package manager. Of course many of the charts will require some extra arguments or configuration files but that is expected.

You can manage your repos using `helm repo`.

```text
$ helm repo list
NAME    URL
stable  https://kubernetes-charts.storage.googleapis.com
local   http://127.0.0.1:8879/charts
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

List all the available charts or search for a specific one with `helm search`.

```text
$ helm search gitlab
NAME                    VERSION DESCRIPTION
stable/gitlab-ce        0.1.11  GitLab Community Edition
stable/gitlab-ee        0.1.10  GitLab Enterprise Edition
```

For my first try I decided to install something simple like RabbitMQ, I choose the `stable/rabbitmq` chart.

```text
$ helm install stable/rabbitmq
NAME:   winsome-quoll
LAST DEPLOYED: Wed Sep  6 00:24:20 2017
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Secret
NAME                    TYPE    DATA  AGE
winsome-quoll-rabbitmq  Opaque  2     5s

==> v1/PersistentVolumeClaim
NAME                    STATUS  VOLUME                                    CAPACITY  ACCESSMODES  STORAGECLASS  AGE
winsome-quoll-rabbitmq  Bound   pvc-fae1769c-9288-11e7-a50c-000d3a212b2e  8Gi       RWO          default       4s

==> v1/Service
NAME                    CLUSTER-IP  EXTERNAL-IP  PORT(S)                                AGE
winsome-quoll-rabbitmq  10.0.63.40  <none>       4369/TCP,5672/TCP,25672/TCP,15672/TCP  4s

==> v1beta1/Deployment
NAME                    DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
winsome-quoll-rabbitmq  1        1        1           0          4s


NOTES:

** Please be patient while the chart is being deployed **

  Credentials:

    echo Username      : user
    echo Password      : $(kubectl get secret --namespace default winsome-quoll-rabbitmq -o jsonpath="{.data.rabbitmq-password}" | base64 --decode)
    echo ErLang Cookie : $(kubectl get secret --namespace default winsome-quoll-rabbitmq -o jsonpath="{.data.rabbitmq-erlang-cookie}" | base64 --decode)

  RabbitMQ can be accessed within the cluster on port 5672 at winsome-quoll-rabbitmq.default.svc.cluster.local

  To access for outside the cluster execute the following commands:

    export POD_NAME=$(kubectl get pods --namespace default -l "app=winsome-quoll-rabbitmq" -o jsonpath="{.items[0].metadata.name}")
    kubectl port-forward $POD_NAME 5672:5672 15672:15672

  To Access the RabbitMQ AMQP port:

    echo amqp://127.0.0.1:5672/

  To Access the RabbitMQ Management interface:

    echo URL : http://127.0.0.1:15672

$ kubectl get pod
NAME                                      READY     STATUS              RESTARTS   AGE
winsome-quoll-rabbitmq-2314639754-695wn   0/1       ContainerCreating   0          16s
$ kubectl get svc
NAME                     CLUSTER-IP   EXTERNAL-IP   PORT(S)                                 AGE
kubernetes               10.0.0.1     <none>        443/TCP                                 5h
winsome-quoll-rabbitmq   10.0.63.40   <none>        4369/TCP,5672/TCP,25672/TCP,15672/TCP   21s
$ kubectl get deployment
NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
winsome-quoll-rabbitmq   1         1         1            0           30s

```

The details for this chart and many more from the stable and the incubator repositories can be found on the Kubernetes [Helm Charts](https://github.com/kubernetes/charts) Github repo. There are a few examples on ACS documentation as well.

Comments are welcome!

--Juanma
