---
layout: post
title: Experimenting with OpenShift and Azure Container Instances
date: 2017-09-21
type: post
published: true
status: publish
categories:
- Containers
- Microsoft
- Red Hat
- Azure
- Cloud
- Cloud-Native
tags:
- Docker
- Azure
- OpenShift
- Containers
- Kubernetes
- Cloud-Native
- Azure Container Instances
- ACI
- ACS
author: juan_manuel_rey
comments: true
---

These days I've playing in my mind with the idea of combining OpenShift and Azure Container Instances. Along wuth teh ACI announcement back in July Microsoft released on GitHub an [ACI Connector for Kubernetes project](https://github.com/Azure/aci-connector-k8s). This project opens the possibility of of deploying ACI from Kubernetes, what it does basically is mimic the Kubelet interface and register it and register itself into the Kubernetes data plane as a node with unlimited capacity. The repo provides a directoy of examples with the YAML file to start playing with it. 

Edit the `aci-connector.yaml` file, add your Azure subscription information (you may need to create a Service Principal if do not want to use an existing one) and deploy it as a pod with `kubectl`

```
$ kubectl create -f aci-connector.yaml
deployment "aci-connector" created
```

This will deploy a pod into the default namespace and will create the `aci-connector` node. 

```
$ kubectl get pod
NAME                             READY     STATUS              RESTARTS   AGE
aci-connector-1703127997-krgb7   0/1       ContainerCreating   0          24s
constraintpod                    1/1       Running             0          13d
sise-2343179185-p7xzv            1/1       Running             0          13d
twocontainers                    2/2       Running             120        13d
$
$ kubectl get node
NAME                    STATUS                     AGE       VERSION
aci-connector           Ready                      45s       v1.6.6
k8s-agent-4ec80835-0    Ready                      15d       v1.6.6
k8s-agent-4ec80835-1    Ready                      15d       v1.6.6
k8s-agent-4ec80835-2    Ready                      15d       v1.6.6
k8s-master-4ec80835-0   Ready,SchedulingDisabled   15d       v1.6.6
```

After that deploy a pod into your Kubernetes cluster with `nodeName: aci-connector` in the container spec, the repo also has an NGINX example.

```
$ kubectl create -f nginx-pod.yaml
pod "nginx" created
$ kubectl get pod
NAME                             READY     STATUS    RESTARTS   AGE
aci-connector-1703127997-krgb7   1/1       Running   0          2m
constraintpod                    1/1       Running   0          13d
nginx                            1/1       Running   0          26s
sise-2343179185-p7xzv            1/1       Running   0          13d
twocontainers                    2/2       Running   120        13d
```

And if you check your container instance in Azure you will see a new `nginx` container there. 

```
$ az container list
Name        ResourceGroup    ProvisioningState    Image                     IP:ports          CPU/Memory       OsType    Location
----------  ---------------  -------------------  ------------------------  ----------------  ---------------  --------  ----------
nginx       acidemorg        Succeeded            nginx                     13.64.244.205:80  1.0 core/1.5 gb  Linux     westus
helloworld  acidemorg        Succeeded            microsoft/aci-helloworld  52.174.45.71:80   1.0 core/1.5 gb  Linux     westeurope
```

With all of this in mind I decided to try the same exercise on the OpenShift Origin cluster I have on Azure internal subscription. But first remember that this is an experiment, **none of this is supported by Microsoft or Red Hat**. 

As I was expecting the first try failed miserably, the pod kept restarting and the `aci-connector` node never appears, the pod logs shouted this errors.

```
$ oc logs -f pod/aci-connector-4041558504-3zj08
${ACI_REGION} not specified, defaulting to "westus"
(node:1) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): Error: EACCES: permission denied, open '/app/tokenstore.json'
(node:1) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
```

I quickly figured out the issue, the ACI connector pod needs to run as a privileged container since OpenShift security is much more strigent that plain Kubernetes. I needed to create a Service Account aith `cluster-admin` role in order to be able to register the new node in the clusters and modify the YAML file to add the Security Context.  

Using `oc` utility create the new Service Account and the with `oadm` give it the proper permissions. 

```
oc create serviceaccount aci-connector
oadm policy add-cluster-role-to-user cluster-admin system:serviceaccount:default:aci-connector
oadm policy add-scc-to-user privileged system:serviceaccount:default:aci-connector
```

After this modify `aci-connector.yaml` to look like this.

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: aci-connector
  namespace: default
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: aci-connector
    spec:
      serviceAccount: aci-connector
      containers:
      - name: aci-connector
        image: microsoft/aci-connector-k8s:latest
        imagePullPolicy: Always
        env:
        - name: AZURE_CLIENT_ID
          value: <redacted>
        - name: AZURE_CLIENT_KEY
          value: <redacted>
        - name: AZURE_TENANT_ID
          value: <redacted>
        - name: AZURE_SUBSCRIPTION_ID
          value: <redacted>
        - name: ACI_RESOURCE_GROUP
          value: acidemorg
        securityContext:
          privileged: true
```

Now proceed to deploy `aci-connector` with `oc`, the OpenShift client.

```
$ oc create -f aci-connector.yaml
deployment "aci-connector" created
$
$ oc get node
NAME                      STATUS                     AGE
aci-connector             Ready                      21m
origin-cluster-infra-0    Ready                      77d
origin-cluster-master-0   Ready,SchedulingDisabled   77d
origin-cluster-node-0     Ready                      77d
origin-cluster-node-1     Ready                      77d
```

YAY! This time the deployment worked and the `aci-connector` node appeared. Next step, deploy the NGINX example and here is where I hit a wall... the pod got stuck into `Pending` status.

```
$ oc get pod/nginx
NAME      READY     STATUS    RESTARTS   AGE
nginx     0/1       Pending   0          17m
```

The logs showed an error from the server about the address

```
$ oc logs -f pod/nginx
Error from server: no preferred addresses found; known addresses: []
$ oc get pod/nginx -o wide
NAME      READY     STATUS    RESTARTS   AGE       IP        NODE
nginx     0/1       Pending   0          44m       <none>    aci-connector
```

After digging a bit more and discussing this with a couple of colleagues and the ACI folks I found an error on the `aci-connector` pod logs. 

```
Error: The server '172.30.182.251:5000' in the 'imageRegistryCredentials' of container group 'nginx' is invalid. It should be a valid host name without protocol.
```

And much more errors that in the end pointed me to what I believe is at least one of the issues. OpenShift comes with its own container registry, used amongst other tasks for for S2I (Source to Image), and ACI tried to pull the image from it and obviously cannot reach it since is not exposed to the outside world. The good thing is that OpenShift supports external registries as well, and this basically will lead me to my next experiment which is trying to integrate OpenShift with Azure Container Registry. 

Will publish a new post soon about the integration with ACR and a second post about my experiments with ACI and OpenShift. In the meantime all kind of suggestions are welcome so please leave a comment or reach me on Twitter. 

--Juanma

