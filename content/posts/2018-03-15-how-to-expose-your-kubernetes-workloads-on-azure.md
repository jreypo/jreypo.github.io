---
title: How to expose your Kubernetes workloads on Azure
date: 2018-03-15
tags:
- aks
- azure
- cloud-native
- containers
- docker
- kubernetes
- networking
showComments: true
---

In the article about [Kubernetes on ACS]({{< ref "posts/2017-09-05-kubernetes-on-azure-with-azure-container-service.md" >}}) I briefly touched the topic of Kubernetes Ingress, originally I was going to made a post about Ingress however I thought it would be better to explain the different methods to expose a Kubernetes based app and how are they implemented on Azure.

The way to expose your app is by using a Kubernetes Service. There are three types of services, or `ServiceTypes`.

- `ClusterIP`
- `NodePort`
- `LoadBalancer`

And there is also another special way of exposing your workloads known as **Kubernetes Ingress**. I will briefly touch the first two and go deeper into `LoadBalancer` and Ingress since those are more relevant for Azure.

## `ClusterIP ServiceType`

`ClusterIP` is the default Kubernetes `ServiceType`. When a service is set to `ClusterIP` Kubernetes wil expose the service on an cluster internal IP and the service will only be reachable within the cluster. All the Kubernetes services related to the cluster are exposed as `ClusterIP` types.

Lets a see a quick example, we will deploy an NGINX server and exposed it with a `ClusterIP` type service.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
```  

Deploy it with `kubectl` and check the pods and service created.

```text
$ kubectl get pods
NAME                       READY     STATUS    RESTARTS   AGE
my-nginx-5d69b5ff7-4mdcq   1/1       Running   0          1m
my-nginx-5d69b5ff7-c9xxc   1/1       Running   0          1m
$ kubectl get service
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.0.0.1      <none>        443/TCP   131d
my-nginx     ClusterIP   10.0.199.84   <none>        80/TCP    1m
```

In an Azure deployed cluster this type of service will be used by inter-cluster service communication like in any other cluster and it will not be reacheable from the outside even if we use the [Azure CNI plugin](https://github.com/Azure/azure-container-networking) for container networking.

## `NodePort ServiceType`

The `NodePort` service is no more than a port open in every node of the cluster, Kubernetes will take care of routing the incoming traffic to the service. The port will be allocated by the master(s) from a cluster configured pool, the default pool is 30000-32676. Take a look at the below diagram.

[![](/images/K8S-NodePort-Service.png "Kubernetes NodePort service")](/images/K8S-NodePort-Service.png)

Of course `NodePort` services are supported in any Kubernetes cluster running on Azure, including those deployed with ACS, ACS Engine or AKS. However since the nodes aren't exposed directly to the internet the functionality is very limited, also using a `NodePort` service for a production workload is not the best choice due to its limitations, however it can be used as building block for Ingress or a `LoadBalancer` service.

## `LoadBalancer ServiceType`

The `LoadBalancer` service is the standard and most common way of exposing your services to the outside world from a Kubernetes cluster running on top a cloud infrastructure like Microsoft Azure. This `ServiceType` will leverage the cloud provider built-in mechanism to provision and configure a load balancer. On Azure this will provision an [Azure Load Balancer](https://docs.microsoft.com/en-us/azure/load-balancer/) and configure the load balancing rules, health probes, backend pools and frontend IPs.

The diagram below illustrates this example with two different apps being exposed to the outside via a `LoadBalancer` service that provisions an Azure Load Balancer with its corresponding public IP address.

[![](/images/K8S-LoadBalancer-Service.png "Kubernetes service")](/images/K8S-LoadBalancer-Service.png)

The load balancer configuration can also be verified in the Azure portal.

[![](/images/kubernetes-lb-azure.png "Azure Load Balancer for Kubernetes services")](/images/kubernetes-lb-azure.png)

This doesn't mean that a new Azure LB is created for each service, as shown in the example AKS will deploy an Azure Load Balancer and publish all the services through it until it reaches the maximum allowed vaulues in terms of front'end IPs, rules, etc, and then it will provision a new Azure Load Balancer instance. For example in one of my AKS clusters I have three services exposed through the same load balancer instance.

```text
$ az network lb list
Location       Name                    ProvisioningState    ResourceGroup                           ResourceGuid
-------------  ----------------------  -------------------  --------------------------------------  ------------------------------------
westeurope     k8s-master-lb-73809577  Succeeded            k8s-aad-demo                            a5b483c6-0e16-49ef-b5ac-db2b60bcc4a6
westeurope     vmss1lb                 Succeeded            vmssrgtest                              234ac3b4-1bd1-43f8-92fd-c46399163aa8
canadacentral  kubernetes              Succeeded            MC_aksrg-ready_aks-ready_canadacentral  84686a77-b527-48d3-bfcf-1b51490966ff
$ az network lb frontend-ip list -g MC_aksrg-ready_aks-ready_canadacentral --lb-name kubernetes
Name                              PrivateIpAllocationMethod    ProvisioningState    ResourceGroup
--------------------------------  ---------------------------  -------------------  --------------------------------------
ad0fbc671079611e88e9c0a58ac1f0b1  Dynamic                      Succeeded            MC_aksrg-ready_aks-ready_canadacentral
ad0fe9226079611e88e9c0a58ac1f0b1  Dynamic                      Succeeded            MC_aksrg-ready_aks-ready_canadacentral
a6852cdb8079811e88e9c0a58ac1f0b1  Dynamic                      Succeeded            MC_aksrg-ready_aks-ready_canadacentral
$ az network lb rule list -g MC_aksrg-ready_aks-ready_canadacentral --lb-name kubernetes
  BackendPort  EnableFloatingIp      FrontendPort    IdleTimeoutInMinutes  LoadDistribution    Name                                       Protocol    ProvisioningState    ResourceGroup
-------------  ------------------  --------------  ----------------------  ------------------  -----------------------------------------  ----------  -------------------  --------------------------------------
           80  True                            80                       4  Default             ad0fbc671079611e88e9c0a58ac1f0b1-TCP-80    Tcp         Succeeded            MC_aksrg-ready_aks-ready_canadacentral
         7744  True                          7744                       4  Default             ad0fe9226079611e88e9c0a58ac1f0b1-TCP-7744  Tcp         Succeeded            MC_aksrg-ready_aks-ready_canadacentral
           80  True                            80                       4  Default             a6852cdb8079811e88e9c0a58ac1f0b1-TCP-80    Tcp         Succeeded            MC_aksrg-ready_aks-ready_canadacentral
$ kubectl get service
NAME                    TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
brigade-brigade-api     ClusterIP      10.0.210.50    <none>          7745/TCP         5d
brigade-brigade-cr      LoadBalancer   10.0.151.0     52.228.64.137   80:32139/TCP     5d
brigade-brigade-gw      LoadBalancer   10.0.214.191   52.228.26.152   7744:31483/TCP   5d
example-python-python   ClusterIP      10.0.53.32     <none>          80/TCP           5d
kashti-kashti           LoadBalancer   10.0.83.80     52.237.35.43    80:32137/TCP     5d
kubernetes              ClusterIP      10.0.0.1       <none>          443/TCP          5d
```

The same procedure will be repeated for each app or api that needs external access, for a few of them it can work but in a public cloud provider the public IP addresses are not free and also you will probably exhaust the maximum allowed number public IP addresses, in Azure for example the default limit is 20 public IP addresses per subscription. There is another technical restriction in Azure, the standard Azure Load Balancer is not able to do SSL termination. If you are going to run just a few services then `LoadBalancer` is perfect solution, and tightly integrated on Azure, however for more complex deployments the best option is to use Kubernetes Ingress.

## Internal Load Balancer

By default a `LoadBalancer` service will provision a public Azure Load Balancer, however an internal one can also be provisioned in order to expose the service to other internal workloads running on you Azure subscription(s). Keep in mind that although this configuration will work with any Kubernetes installation on Azure is only useful with manually or ACS Engine deployed clusters since at this moment custom VNET and VNET Peering are not supported with AKS.

To provision an internal `LoadBalancer` we will need to add the following annotations to the service.

```yaml
metadata:
  name: my-nginx
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
```

Lets create the example NGINX deployment with `LoadBalancer` service and the internal annotations.

```text
$ kubectl create -f nginx-svc-ilb.yaml
service "my-nginx" created
deployment "my-nginx" created
$ kubectl get pods
NAME                       READY     STATUS    RESTARTS   AGE
my-nginx-5d69b5ff7-wwcvp   1/1       Running   0          7m
my-nginx-5d69b5ff7-z5xfp   1/1       Running   0          7m
$ kubectl get service
NAME         TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)        AGE
kubernetes   ClusterIP      10.0.0.1      <none>         443/TCP        143d
my-nginx     LoadBalancer   10.0.39.204   10.240.0.127   80:31052/TCP   7m

```

The field `EXTERNAL-IP` shows an internal IP address belonging to the same subnet as the nodes of the cluster. Take a look into the Azure portal and verify the new internal load balancer has been created.

[![](/images/kubernetes-ilb-azure.png "Azure Internal Load Balancer for Kubernetes services")](/images/kubernetes-ilb-azure.png)

## Kubernetes Ingress

Ingress is by far the most interesting and powerful of all the methods available in Kubernetes to expose your services. In the [Kubernetes official documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress is defined as *An API object that manages external access to the services in a cluster, typically HTTP*. Ingress can provide load balancing, SSL termination and name-based virtual hosting. However in the real sense Ingress is not a service but a construct that sits on top of your services as an entry point to the cluster, providing simple host and URL based HTTP routing capabilities.

[![](/images/K8S-Ingress-Service.png "Kubernetes Ingress")](/images/K8S-Ingress-Service.png)

Ingress has two main components:

- Ingress Resource
- Ingress Controller

Defined by namespace, the Ingress resource represents the configured routing rules that will govern how the services are accessed from the external user. Using the example from the diagram, if a request for `contoso.com` reaches the cluster then ingress will route the traffic to `Service` but if the request is for `contoso.com/accounting` then ingress will route the traffic to Service D. Following is a common Ingress resource definition.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: contoso.com
    http:
      paths:
      - backend:
          serviceName: contoso-service
          servicePort: 80
  - host: fabrikam.net
    http:
      paths:
      - path:
        backend:
          serviceName: fabrikam-service
          servicePort: 80
```

Deployed as a pod the Ingress Controller will route the traffic to the corresponding service following the rules defined on the Ingress Resource. On top of Ingress a `LoadBalancer` service needs to be deployed in order to expose the Ingress controller to the outside, this will be the main entry point into the cluster.

There are several Ingress Controllers available to be used on Azure.

- [NGINX Ingress Controller](https://github.com/kubernetes/ingress-nginx)
- [Heptio Contour](https://github.com/heptio/contour)
- [Traefik](https://traefik.io/)

The most common, and almost the de-facto standard in Kubernetes, is NGINX. It is also the [recommended one right now for AKS](https://docs.microsoft.com/en-us/azure/aks/ingress). NGINX is the only one I've used, but I am dying to try the other two specially Contour so expect a future blog post on it. For Traefik I found [this blog post](http://hypernephelist.com/2017/10/17/getting-started-with-traefik-and-k8s-using-acs.html) describing how to use with Azure Container Service.

## Deploying Ingress with NGINX Ingress Controller

The installation of Ingress is very well described on [AKS documentation](https://docs.microsoft.com/en-us/azure/aks/ingress) but I will briefly describe the process for the sake of completion of the article. Thanks to [Helm](https://helm.sh/) deploying an Ingress Controller on your Kubernetes cluster is is as easy as:

```text
helm install stable/nginx-ingress
```

The installation will deploy two pods for the NGINX contoller and two services, one of them of `LoadBalancer` type.

```text
$ kubectl get pods -o wide
NAME                                                 READY     STATUS             RESTARTS   AGE       IP            NODE
nginx-ingress-controller-5b4b998b67-mlkqj            1/1       Running            0          30d       10.244.5.23   aks-nodepool1-68968121-1
nginx-ingress-default-backend-58bf6f478b-zmj79       1/1       Running            0          30d       10.244.3.16   aks-nodepool1-68968121-3
$ kubectl get service -o wide
NAME                            TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE       SELECTOR
kubernetes                      ClusterIP      10.0.0.1       <none>          443/TCP                      41d       <none>
nginx-ingress-controller        LoadBalancer   10.0.126.42    52.237.39.132   80:30930/TCP,443:31914/TCP   30d       app=nginx-ingress,component=controller,release=nginx-ingress
nginx-ingress-default-backend   ClusterIP      10.0.56.240    <none>          80/TCP                       30d       app=nginx-ingress,component=default-backend,release=nginx-ingress
```

Create now the Ingress Resource, I am using the above minimal example.

```text
$ kubectl create -f ingress-example.yml
ingress "my-ingress" created
$ kubectl get ingress
NAME         HOSTS                          ADDRESS   PORTS     AGE
my-ingress   www.contoso.com,fabrikam.net             80        9s
$
$ kubectl describe ingress my-ingress
Name:             my-ingress
Namespace:        default
Address:
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host             Path  Backends
  ----             ----  --------
  www.contoso.com
                      contoso-service:80 (<none>)
  fabrikam.net
                      fabrikam-service:80 (<none>)
Annotations:
Events:
  Type    Reason  Age   From                Message
  ----    ------  ----  ----                -------
  Normal  CREATE  25s   ingress-controller  Ingress default/my-ingress
```

If you want to learn more about Ingress I highly recommend you to read the official Kubernetes Ingress documentation and the docs section of `nginx-ingress` GitHub repository.

We are done for now, in a future post I will touch the topic of deploying and using Contour on AKS.

-- Juanma
