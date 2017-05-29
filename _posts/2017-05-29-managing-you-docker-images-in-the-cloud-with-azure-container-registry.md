---
layout: post
title: Managing you Docker images in the cloud with Azure Container Registry
date: 2017-05-29
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
- Docker
- Azure
- ACS
- Azure Container Service
- Azure Container Registry
- Containers
- Cloud-Native
author: juan_manuel_rey
comments: true
---

Past April  Microsoft [announced the general availability](https://azure.microsoft.com/en-us/blog/azure-container-registry-now-generally-available/) of [Azure Container Regitry](https://azure.microsoft.com/en-us/services/container-registry/). Since I will be using ACR during my next posts about ACS and Kubernetes I tought it would be helpful to write a quick intro about it.

Azure Container Registry, or ACR, is an offering from Microsoft to manage and store Docker container images on Azure, compatible with Docker Registry v2 specification. The service can be managed using Azure Portal, Azure PopwerShell or Azure CLI and the images can be managed using standard Docker tools. And can be easily integrated within an existing CI/CD pipeline. 

- ACR is Docker Registry V2 compatible.
- Allows to store Docker images from a cluster deployed with Azure Container Service, a manually deployed container cluster on Azure,a  developer workstation or from an on-prem cluster.
- Supports Windows and Linux container images. It also has integration with other Azure services that supports Docker images like Service Fabric and App Service.
- Azure Active Directory integration for the authentication. 

One of the advantages of using ACR with your ACS clusters or Azure services is the low latency for the pulling and pushing operations. Lets see how can we create and manage our Docker registry in Azure.

# Create your first registry

As always create your resource group.

```
az group create -n acr-rg -l westeurope
```

Now create the registry, provide the name for the registry, the resource group and the SKU which as of today is just `Basic`.

```
az acr create -n registryprod -g acr-rg -l westeurope --sku Basic
```

This operation will produce two resources in Azure.

- The registry, behind the scenes a Docker registry has been deployed.
- One Storage Account to store the images.

```
$ az resource list -g acr-rg
Name                ResourceGroup    Location    Type                                    Status
------------------  ---------------  ----------  --------------------------------------  --------
registryprod        acr-rg           westeurope  Microsoft.ContainerRegistry/registries
registryprod105019  acr-rg           westeurope  Microsoft.Storage/storageAccounts
```

The registry is automcatically assigned a fqdn with the form `<registry_name>.azurecr.io`.

# Authentication

Te registy is created but right now is not usable since there is no way to authenticate. ACR by default does not even have an admin user, although it can be enabled during the `create` task it can also be enabled afterwards as we will see now. 

```
$ az acr update -n registryprod --admin-enabled true
NAME          RESOURCE GROUP    LOCATION    LOGIN SERVER             CREATION DATE                     ADMIN ENABLED
------------  ----------------  ----------  -----------------------  --------------------------------  ---------------
registryprod  acr-rg            westeurope  registryprod.azurecr.io  2017-05-25T10:50:59.194729+00:00  True
$ az acr credential show -n registryprod
USERNAME      PASSWORD                          PASSWORD2
------------  --------------------------------  --------------------------------
registryprod  SUPER_AWESOME_PASSWORD            ANOTHER_SUPER_AWESOME_PASSWORD
```

Having an admin account is useful however the best option is to use a service account in the form of a Service Principal. A Service Principal is an account used by cloud apps and services to access Azure APIs. The good thing about Service Principals is that they are backed by Azure Active Directory meaning you can audit and revoke the account at any time. 

You can create a new Service Principal, assigning the proper role and scope over ACR during the operation. The roles can be `Owner`, `Contributor` or `Reader`. The first one will give the SP account full control over ACR, the second will allow to push and pull images and the last one will have read-only permissions allowing the SP account just to pull Docker images from the registry. 

```
az ad sp create-for-rbac --scopes /subscriptions/11111111-1111-1111-1111-1111111111111/resourcegroups/acr-rg/providers/Microsoft.ContainerRegistry/registries/registryprod --role Owner --password <password>
```

Alternatively you can use an existing service principal and assign access to your registry.

```
az role assignment create --scope /subscriptions/11111111-1111-1111-1111-1111111111111/resourcegroups/acr-rg/providers/Microsoft.ContainerRegistry/registries/registryprod --role Owner --assignee <app-id>
```

Test the authentication from your Docker host with `docker login`.

```
azureuser@coreos ~ $ sudo docker login registryprod.azurecr.io -u <service_principal_id> -p <service_principal_key>
Login Succeeded
azureuser@coreos ~ $
```

# Image Management 

Azure CLI allows you to perform some basic tasks for the images stored in a registry.

List the repositories currently in the registry. 

```
$ az acr repository list -n registryprod
Result
--------------------
baseimages/alpine
baseimages/fedora
testimages/nginx
testimages/wordpress
webserver/nginx
webservers/nginx
```

Retrieve the tags of a given registry. 

```
$ az acr repository show-tags -n acrinfra1 --repository baseimages/fedora
Result
--------
24
latest
```

## Removing images

While pushing, pulling and tagging images are non-trivial tasks performed from `docker` client, removing an image can be a bit tricky. Currently there is now way to easily remove an image from ACR, yes Docker registry v2 has a [delete API](https://docs.docker.com/registry/spec/api/#deleting-an-image) but unfortunately is not exposed on ACR registries. 

However there is a method to remove an image from ACR but the caveat is that it must be done from the Azure Portal and the user must have the Owner role either for the registry or the whole subscription.

From Azure Portal access the storage account backing up your registry and then select Blobs.

[![](/images/acr_delete_image_1.png)]({{site.url}}/images/acr_delete_image_1.png)

Inside Blobs navigate through the different folders until you get to Respositories. **\<no-name\> -> docker -> registry -> v2 -> repositories**. In repositories select one, in my case baseimages.

[![](/images/acr_delete_image_2.png "Select repository")]({{site.url}}/images/acr_delete_image_2.png)

Inside the repository select the container image, I choose fedora.

[![](/images/acr_delete_image_3.png "Select container image")]({{site.url}}/images/acr_delete_image_3.png)

Then click on **_manifests** and inside it select **tags**. The different tags for the image will be listed.

[![](/images/acr_delete_image_4.png "Select the tag")]({{site.url}}/images/acr_delete_image_4.png)

Select the tag and then click on **Delete folder** at the top bar. 

[![](/images/acr_delete_image_5.png "Delete folder")]({{site.url}}/images/acr_delete_image_5.png)

If we go back to Azure CLI and list the tags for `baseimages/fedora` the deleted one will not show up. 

```
$ az acr repository show-tags -n acrinfra1 --repository baseimages/fedora
Result
--------
latest
```

-- Juanma