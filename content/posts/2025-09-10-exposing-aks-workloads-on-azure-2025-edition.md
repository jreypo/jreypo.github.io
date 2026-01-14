---
title: Exposing AKS Workloads on Azure - 2025 Edition
date: 2025-09-10 12:15:00 +0100
categories:
- Microsoft
- Azure
tags:
- Microsoft
- Kubernetes
- Cloud-Native
- Nginx
- AKS
showComments: true
---

In cloud-native architecture, getting traffic into your Kubernetes cluster is critical. Back in 2018 I published an article in the blog about [how Kubernetes workloads could be exposed on Azure]({{< ref "posts/2018-03-15-how-to-expose-your-kubernetes-workloads-on-azure.md" >}}). Since then, Azure Kubernetes Service (AKS) and Kubernetes networking have evolved significantly, introducing new ingress controllers and load balancing features. This updated guide revisits how to expose your AKS-hosted workloads, comparing **modern ingress options** (like NGINX, Azure Application Gateway Ingress Controller, and the new Gateway API) and detailing **load-balancing strategies** (external vs. internal, L4 vs L7).

## Service Types and Load Balancers in AKS

**Kubernetes Service Types:** In Kubernetes, you typically expose applications via a [Service](https://kubernetes.io/docs/concepts/services-networking/service/). The main Service types are:

- **ClusterIP** – *internal-only*. Default type; accessible only within the cluster (no external access).
- **NodePort** – *opens a port on each node*. Allows access via `<NodeIP>:NodePort`. Useful as a building block (e.g. for ingress controllers) but not ideal for direct production use, since nodes themselves aren’t directly internet-facing by default.
- **LoadBalancer** – *cloud provisioned load balancer*. This is the standard way to expose services externally on Azure. The AKS cloud integration will **provision an Azure Load Balancer (L4)** with a public IP, and map inbound traffic to your pods.
- **ExternalName** - *maps a service to an external DNS name*. Returns a CNAME record pointing to the specified hostname instead of routing to cluster IPs; useful for referencing services outside the cluster without hardcoding their addresses.

**Azure Load Balancer (L4) Behavior:** With a Service of type `LoadBalancer` on AKS, Azure creates or uses a **Standard Azure Load Balancer** (ILB or external) to forward traffic to the service’s pods. By default, AKS clusters share one outbound load balancer for multiple services, adding multiple front-end IPs and rules as needed (up to Azure’s limits). This means you **won’t get a new load balancer per service** unless the existing one runs out of capacity. Still, each Service **does consume a public IP** (unless using an internal LB) and Azure has default limits (e.g. 20 public IPs per region per subscription). Relying on many external load balancers can quickly hit these limits and incur costs for each IP. Moreover, an L4 load balancer **only forwards traffic**; it cannot handle SSL termination or any HTTP routing like path-based dispatch.

**Internal vs. External Load Balancers:** By default, a `LoadBalancer` service in AKS is **external** – it gets a public IP. But you can also create an **internal** load balancer (for private-only access within an Azure virtual network or via peering). To do this in AKS, add an annotation `service.beta.kubernetes.io/azure-load-balancer-internal: "true"` to the Service. This instructs AKS to allocate an **internal IP** from the cluster’s virtual network, instead of a public IP. The example below shows a service manifest for an internal load balancer:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-app
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: internal-app
```

In this example, the service will receive an internal IP (visible in the `EXTERNAL-IP` field as a private address). AKS creates an internal Azure Load Balancer in the node resource group and attaches that IP. Internal load balancers let you expose services privately to consumers in the same VNet (or peered VNets) – for example, exposing an internal API to other Azure services or on-prem via VPN.

**Important:** AKS now uses Standard SKU load balancers by default. (Basic SKU is deprecated and support will retire by Sept 30, 2025. Standard LB supports both Public and Internal modes; it also offers features like Outbound Rules for egress and configurable idle timeouts. Ensure your cluster is using Standard SKU for production.

## Kubernetes Ingress: Layer-7 Routing for Services

While Services with type `LoadBalancer` handle direct L4 traffic, Kubernetes **Ingress** provides a higher-level, layer-7 entry point. According to Kubernetes docs, *Ingress is an API object that manages external access to services in a cluster, typically HTTP/HTTPS*. In practice, an Ingress allows you to define **virtual hosts, URL path routing, TLS termination, and more** – all within the cluster, instead of configuring these on an external load balancer. For example, you can route `api.starlabs.lab/*` to one service and `starlabs.lab/web/*` to another, using a single IP address. Ingress **consolidates access** to multiple services behind a single endpoint (or a few endpoints), which is more efficient for large apps than having dozens of L4 load balancers.

However, an **Ingress by itself is only a set of rules**. The Kubernetes control plane doesn’t actually implement those rules – you need an **Ingress Controller** (a controller/service that watches Ingress objects and configures a proxy or load balancer accordingly). The Ingress Controller is typically a **pod running in the cluster** (for example, an NGINX or Envoy proxy) or an integration with an external load balancer (like Azure Application Gateway). The controller is the component that actually **routes incoming HTTP(S) traffic** per the Ingress rules.

**Relationship between Ingress and LoadBalancer:** Even when using ingress, you still need a way to get traffic into the cluster to reach the ingress controller. Usually, that means you deploy a Service of type `LoadBalancer` in front of the Ingress Controller pod(s) (or use an external L7 appliance). In other words, the chain is: External client → Azure Load Balancer (L4) → Ingress Controller (listening on node port or host network) → Cluster services. The ingress controller then applies the routing rules. The only case you might not need a separate L4 LB is if the ingress controller itself is an external service (as with AGIC, which uses the Azure Application Gateway as the entry point). We will discuss those scenarios.

**Ingress Controller Options:** Many ingress controllers have emerged. Back in 2018, the common choices on Azure were *NGINX Ingress Controller*, *Heptio Contour* (Envoy-based), or *Traefik*. Today in 2025, AKS supports an even richer set of ingress solutions:

1. **NGINX Ingress Controller (OSS or Managed Add-on)** – The venerable open-source NGINX-based controller, now available as an AKS-managed add-on for ease of use.
2. **Azure Application Gateway Ingress Controller (AGIC)** – An AKS add-on that configures an external Azure Application Gateway to act as ingress. Integrates Azure’s native L7 load balancer (with WAF) with your cluster.
3. **Kubernetes Gateway API (with Application Gateway for Containers)** – A new approach using Kubernetes Gateway API resources and Azure’s *Application Gateway for Containers (AGC)* service (the evolution of AGIC). Offers advanced routing and is poised to become the primary ingress mechanism in the future.
4. **Istio Ingress Gateway** – Part of the AKS **Istio service mesh add-on** (introduced in recent years). Istio’s Envoy-based ingress gateway can be used for HTTP(S) ingress, especially if you’re adopting service mesh for traffic management.
5. **Other Third-Party Controllers** – e.g. *Traefik*, *HAProxy*, *Kong*, *Contour* etc. These can be user-installed on AKS. They may offer unique features (Traefik integrates with Let’s Encrypt, Kong provides API gateway features, Contour (Envoy) supports Gateway API, etc.). These are not managed by Azure, but remain viable options depending on your needs.

Below we dive into the major options (NGINX, AGIC, AGC/Gateway API, etc.), and then compare their features and usage.

## NGINX Ingress Controller (with AKS Application Routing Add-on)

**NGINX Ingress** has been the workhorse of Kubernetes ingress for years. It runs NGINX (or NGINX Plus) as a reverse-proxy pod in the cluster to implement Ingress resources. In AKS, you can either deploy the standard open-source NGINX Ingress Controller yourself (via Helm or manifests), [**or use the AKS-managed add-on**](https://learn.microsoft.com/en-us/azure/aks/app-routing) called *“HTTP Application Routing”* which now leverages NGINX under the hood. The managed add-on greatly simplifies setup: you can enable it with an `az aks create --enable-app-routing` flag or on an existing cluster, and AKS will install and manage the NGINX ingress controller for you.

**Features:** The managed NGINX add-on provides *easy configuration of a production-grade ingress*. It automatically sets up an external DNS name and Azure DNS zone for your ingress if desired, and can pull TLS certificates from Azure Key Vault for SSL termination. This reduces manual work compared to a DIY NGINX setup. NGINX itself supports **path-based and host-based routing, TLS offload, HTTP to HTTPS redirects, rewrites, etc.** via annotations or ConfigMap settings. It’s a reliable choice for typical web applications. However, because it’s essentially the open-source NGINX, advanced features like WAF, or sophisticated routing (based on HTTP headers, geolocation, etc.) are not built-in (you’d integrate external tools or use NGINX Plus for some of that).

**Deployment and Config:** If you use the AKS add-on, enabling it will create an ingress controller deployment in a dedicated namespace (e.g. `app-routing-system`) and an `IngressClass` named `webapprouting.kubernetes.azure.com`. You then create standard Ingress resources in your apps’ namespaces with that class. If you install NGINX manually instead, you’ll typically use the class name “`nginx`”. In either case, you will also have a Service of type LoadBalancer fronting the NGINX pods (the add-on sets this up for you automatically). By default this is a public LoadBalancer, but you can convert it to internal by adding the internal annotation to the NGINX service.

Here’s an example **Ingress resource** for NGINX, illustrating host and path routing:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  ingressClassName: nginx                # or "webapprouting.kubernetes.azure.com" for AKS add-on
  rules:
  - host: www.starlabs.lab
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: starlabs-service
            port:
              number: 80
```

This Ingress defines that HTTP requests to `www.starlabs.lab/` will be forwarded to the Kubernetes Service `starlabs-service` on port 80. We could add more rules for other hosts or paths. Once applied, the NGINX controller will configure its NGINX instance accordingly. (Remember, we must have an external IP for the NGINX controller itself – AKS will show that via `kubectl get ingress` (address) or the `nginx` Service’s external IP). You can then point a DNS CNAME or A record to that IP. If using the AKS add-on with DNS integration, it can even manage a DNS name for you in Azure DNS.

**Internal Ingress:** To expose ingress only privately, deploy the NGINX controller’s Service as internal. For instance, with the add-on, you can follow AKS documentation to [configure NGINX ingress controller to support Azure private DNS zones](https://learn.microsoft.com/en-us/azure/aks/create-nginx-ingress-private-controller) – essentially it involves setting the internal LB annotation and using Azure Private DNS for name resolution. This way, you get an internal IP for NGINX and a DNS name that resolves only inside your network.

**Scaling & Considerations:** NGINX Ingress runs inside your AKS cluster, so **its performance depends on cluster resources**. You’ll want to run at least 2 replicas for HA, and monitor CPU/memory usage on the ingress pods under load. NGINX can handle a high throughput, but for extremely large scale you may need to allocate dedicated node pools or tune kernel settings. Also consider that any additional features (WAF, OAuth2 proxy, etc.) would be DIY – as opposed to Azure’s managed offerings which might have those built-in (more on that next). NGINX ingress supports basic authentication, TLS passthrough, and other features through annotations or sidecar containers as needed. It’s a robust choice for many scenarios, but when enterprise requirements like strict security or global routing arise, you might look to the alternatives below.

## Azure Application Gateway Ingress Controller (AGIC)

One major advancement since my original article is the [**Application Gateway Ingress Controller (AGIC)**](https://learn.microsoft.com/en-us/azure/application-gateway/ingress-controller-overview) for AKS. This is an Azure-first approach to ingress. Instead of running a proxy in your cluster for L7 routing, AGIC leverages [**Azure’s Application Gateway**](https://learn.microsoft.com/en-us/azure/application-gateway/) service – a fully managed L7 load balancer that supports features like **autoscaling, path-based routing, cookie affinity, SSL offload, and Web Application Firewall (WAF)**. The AGIC component in your cluster is a small controller pod that watches Ingress resources and configures the Azure Application Gateway accordingly.

**How AGIC Works:** When you enable the AGIC add-on for AKS, you associate an Azure Application Gateway (v2 SKU) with your cluster (either AGIC can create a new one, or use an existing App Gateway via its resource ID). The AGIC runs in your cluster (as a deployment) but consumes minimal resources – its job is to translate Kubernetes Ingress rules into corresponding Azure Application Gateway settings via Azure APIs. The Application Gateway itself sits outside the cluster (in your Azure subscription) and acts as the entry point for HTTP/HTTPS traffic. It will have its own public IP (or DNS name), and it connects **directly to your pods** via private IPs – **no need for Kubernetes NodePort or an Azure L4 load balancer** in front. This eliminates an extra network hop. In fact, one of the touted benefits is that by avoiding the kube-proxy/NodePort path, **App Gateway can reduce ingress latency (by up to \~50%) compared to in-cluster ingress solutions**.

**Features:** Application Gateway (v2) brings a host of features to your ingress: SSL/TLS termination (with certificate management in Azure), end-to-end TLS if needed, **layer-7 routing on hostnames and paths**, the ability to route based on **IP or query string or cookies** (with rewrite rules), active health probing of pods, and crucially a **Web Application Firewall**. By offloading to App Gateway, you also get **autoscaling** of the ingress capacity that’s independent of your AKS nodes – the gateway can scale out under load without consuming any cluster CPU/memory. Application Gateway is a multi-tenant Azure service running in its own subnets, so it doesn’t burden your nodes with proxy workloads.

**Integration with AKS:** AGIC is provided as an AKS managed add-on (enabled with `az aks enable-addons --addons ingress-appgw`). This makes installation very easy – no Helm charts to maintain. The add-on ensures the AGIC pod runs with the correct permissions to talk to Azure ARM and update the App Gateway config. One limitation is that AGIC requires the AKS cluster use **Azure CNI networking** (so pods have VNet IPs that App Gateway can reach) and the Application Gateway must be v2 SKU (Standard_v2 or WAF_v2). Typically, you place the App Gateway in the same virtual network as the AKS cluster (or use VNet peering) so it can reach cluster private IPs.

**Using AGIC (Ingress resources):** You still define Kubernetes Ingress objects (the same way you would for NGINX), but you associate them with an ingress class that AGIC watches. By default, AGIC will watch Ingresses with the class annotation `azure/application-gateway` or `ingressClassName: azure-application-gateway`. For example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app1-ingress
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
    - host: app1.mydomain.com
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: app1-service
              port:
                number: 80
```

AGIC will see this and configure the Azure Application Gateway to route `app1.mydomain.com` traffic to `app1-service`’s pods. If you have multiple Ingresses, it will aggregate all rules onto the single App Gateway (up to its limits of listeners and backends). **No separate Azure LBs or public IPs** are needed for each service – everything funnels through the one Application Gateway IP.

**Benefits and Drawbacks:** The big draw of AGIC is the **integration with Azure’s native capabilities**. You get a fully managed, enterprise-grade L7 load balancer with WAF. For companies already using Application Gateway for other apps, this brings Kubernetes services into that fold. Also, eliminating the in-cluster ingress pods frees those resources for your apps. On the flip side, **Application Gateway has its own costs** (it’s an Azure service running 24/7, billed per capacity unit and hours) – running it might be more expensive than a small OSS ingress on your nodes (depending on scale). Additionally, changes to Application Gateway configuration historically had some latency – the AGIC add-on works to apply updates in near-real-time, but large changes may take a short while to propagate (it’s gotten faster over time). Also note, some Kubernetes ingress features that are easy via annotations in NGINX might not have equivalents in AGIC. You are somewhat constrained by **what App Gateway supports**. For example, App Gateway (prior to 2023) could not do HTTP header-based routing or URL rewrites without additional tricks – those were limitations if you needed them (this is one reason the new AGC was created, as we’ll see).

Finally, **private/internal usage**: Initially, Application Gateway did not support a fully private mode for AKS (the gateway needed a public IP or at least public frontend). However, Azure has introduced *Private Link* and private frontend support for [Application Gateway v2](https://learn.microsoft.com/en-us/azure/application-gateway/private-link). It is now possible to deploy an Application Gateway with only an internal IP (no public exposure). This means you could use AGIC for internal-only scenarios by deploying a private Application Gateway and maybe linking it with Private DNS. (As of late 2023, fully private App Gateway for ingress was still in preview in some regions, but by 2025 this is expected to be broadly available.) In summary, AGIC can handle both internet-facing and internal ingress, but check Azure’s documentation for any region-specific support of internal mode.

**In Practice:** Many AKS users adopt AGIC when they require **strong security (WAF)**, need to reuse an existing Application Gateway (perhaps in a hub network), or want to minimize cluster management of ingress. It’s a cloud-managed solution: Microsoft manages the App Gateway instances, ensures high availability (you can deploy it zone-redundant), and you benefit from Azure’s SLAs. AGIC is a mature solution at this point, used in production by many enterprises.

However, Kubernetes networking is moving forward, and AGIC – which relies on the old Ingress API – has some limitations in flexibility. This is where the **Gateway API** and the next evolution, **Application Gateway for Containers (AGC)**, come into play.

## Gateway API and Application Gateway for Containers (AGC)

The [**Gateway API**](https://gateway-api.sigs.k8s.io/) is a relatively new set of extensible resources (custom resource definitions) in Kubernetes that modernizes ingress and load balancing. It was developed by the SIG-Network community to overcome limitations of the `Ingress` resource. For example, Ingress is HTTP-oriented and relies on annotations for any custom behavior (like TCP ports, header routing, traffic splitting), which can be vendor-specific. Gateway API introduces resources like `GatewayClass`, `Gateway`, `HTTPRoute`, TCPRoute, etc., which allow richer expressiveness. In short, it decouples the definition of “a gateway load balancer” (`Gateway` resource) from the routing rules (`HTTPRoute` resources) and supports more protocols and advanced routing out-of-the-box. Think of Gateway API as a superset that can do what Ingress did and more, in a more Kubernetes-native way.

**Azure’s Implementation – AGC:** With this new API, Azure created **Application Gateway for Containers (AGC)** – a cloud service that is effectively “Application Gateway vNext” to integrate with Gateway API. AGC was in preview in 2023 and became generally available in mid-2025. It is the evolution of AGIC with a new architecture: instead of using the old App Gateway and ARM-based updates, it introduces an **Application Load Balancer (ALB) controller** in the cluster which interacts with a new Azure Application Gateway for Containers instance (sometimes referred to as an “ALB” as well). This new App Gateway for Containers supports both the traditional Ingress resources *and* the new Gateway API resources, but it is built to shine with Gateway API. It also runs an Envoy-based data plane (replacing the older NGINX-based one) for better performance and feature velocity.

**Key Improvements in AGC/Gateway API:**

- **Advanced Routing**: Gateway API and AGC support routing on **HTTP headers, query strings, HTTP methods**, and more, which were not possible with basic Ingress without custom hacks. For example, you could route traffic based on a user-agent header, or do A/B testing with weighted routes (traffic splitting).
- **mTLS and Security**: AGC can perform **mutual TLS authentication** with backend pods (ensuring only trusted AKS pods receive traffic). It also supports end-to-end TLS and TLS re-encryption scenarios more flexibly than before.
- **Retries and Traffic Policies**: The new gateway can do automatic **retries** on failures, and more sophisticated traffic management like timeouts, which weren’t configurable in App Gateway before.
- **Improved Performance Scaling**: AGC’s design moves away from ARM-based updates (which in AGIC could be a bottleneck). It aims for near real-time updates to routing as pods scale in/out. Early info suggests it can handle larger scale (more pods, routes) than AGIC (which had known limits around 100 ingress rules/listeners, etc.).
- **Seamless Integration**: AGC is offered as a managed add-on as well (in preview/GA timeline) so that AKS can handle deploying the ALB controller. You can run AGIC and AGC in parallel on a cluster for migration purposes. The goal is eventually to transition fully to AGC.

From a user perspective, if you opt for AGC, you will deploy a `GatewayClass` (provided by Azure, e.g. `azure-alb-external` for external), then a `Gateway` resource that instantiates an Application Gateway for Containers, and `HTTPRoute` resources to define routing to your services. There is also an `ApplicationLoadBalancer` custom resource in some models to represent the Azure backend. Some of these details are abstracted if you let the add-on manage it.

For example, here’s a snippet of what a Gateway manifest might look like for AGC:

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: example-gateway
  namespace: default
spec:
  gatewayClassName: azure-alb-external
  listeners:
  - name: web
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - group: ""        # core group, Secret
        kind: Secret
        name: tls-cert-secret
    allowedRoutes:
      namespaces:
        from: All
```

This defines an Azure ALB (Application Gateway for Containers) that will listen on HTTPS 443 (termination with a cert stored in a Secret). The `gatewayClassName: azure-alb-external` indicates we want Azure to provision an external-facing gateway (with a public IP). The ALB controller in AKS will either create a new Application Gateway for Containers resource in Azure or use one you’ve pre-provisioned, depending on config. Then you would create one or more HTTPRoute resources that bind to this Gateway (by specifying the Gateway in the HTTPRoute’s `parentRefs`) for each hostname or set of paths you want to expose. Those HTTPRoutes replace the old Ingress objects: they can specify detailed matching conditions (headers, methods, etc.) and point to one or multiple backend services (with weights for traffic splitting).

**Current State (2025):** As of mid-2025, AGC is newly GA. Not all features may be immediately at parity with everything App Gateway had (for example, WAF for Containers might still be in preview, as hinted by documentation). But it already supports the major L7 capabilities and is production-ready for many scenarios. It’s **expected that AGC (Gateway API) will gradually become the recommended default** for AKS ingress, especially as users adopt the Gateway API. Microsoft has provided guidance and even migration tooling to move from AGIC (Ingress) to AGC (Gateway API) with minimal downtime. The AGC can even run concurrently with AGIC on the same Application Gateway during transition.

One thing to note: **Gateway API is newer and can be more complex**. If you have simple needs (just a couple of HTTP rules), using Ingress (perhaps backed by AGIC or NGINX) might be easier. Gateway API shines in complex setups (multiple teams defining routes, advanced traffic policy). It also sets the stage for future integration – e.g., service mesh ingress, multi-cluster ingress, etc., can all potentially use Gateway API consistently.

**Internal AGC:** Similar to AGIC, the new AGC initially supports external front-ends. Fully private usage (internal load balancer mode) is on the roadmap – indeed, *private* Application Gateway for Containers frontends are expected (possibly in preview now). Until that is GA, if you need internal-only ingress with Gateway API, a workaround is to use the Gateway API with an internal Azure Load Balancer service or other proxies. But expect Azure to support internal mode soon, given that private endpoints for App Gateway are now available.

## Other Ingress Options: Istio and Third-Party Controllers

Before we compare features, it’s worth mentioning **Istio**. Istio is a popular open-source service mesh, and Azure offers an [Istio Service Mesh add-on](https://learn.microsoft.com/en-us/azure/aks/istio-about) which includes Istio’s Ingress Gateway (an Envoy proxy) by default. You can use the [Istio Ingress Gateway](https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/) as your cluster ingress point – essentially, it’s another in-cluster controller (Envoy) that you expose with a Service `LoadBalancer`. The benefit of Istio’s ingress is if you are already using Istio for east-west traffic (within cluster), you can use it for north-south ingress as well, and leverage Istio’s rich traffic management (e.g., canary releases, mTLS from ingress to pod, etc.). Istio ingress can also be configured via Gateway API (support is evolving – as of early 2025, Istio supports Gateway API v1beta1 on its ingress gateway experimentally). If you’re not using a service mesh, though, Istio ingress may be overkill.

**Third-party controllers** like [Traefik](https://traefik.io/traefik), [Kong](https://developer.konghq.com/kubernetes-ingress-controller/), [HAProxy](https://www.haproxy.com/documentation/kubernetes-ingress/), [Contour](https://projectcontour.io/) continue to be alternatives. For example, Traefik integrates nicely with Let’s Encrypt for automatic TLS cert issuance, I personally use Traefik in my homelab along with NGINX on my Kubernetes local clusters and with Portainer, Kong acts as an API gateway with plugins for auth, rate limiting, etc. These require you to install and manage them yourself on AKS (and keep them updated). They might not integrate with Azure services (e.g., Key Vault or App Gateway) out of the box. Contour (Envoy) is noteworthy for Gateway API support: it’s one of the reference implementations of Gateway API in open source. If you prefer not to use Azure’s AGC, you could deploy Contour and use Gateway API with it on AKS. Just keep in mind support and operations would be up to you.

## Comparison of Ingress and Load Balancing Options

Let’s summarize and compare these options in terms of capabilities:

| **Capability**                 | **Azure LB (Service)**                                                 | **NGINX Ingress**                                                                       | **App Gateway (AGIC)**                                                              | **App Gateway for Containers (AGC)**                                   |
| ------------------------------ | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Layer**                      | L4 (TCP/UDP)                                                           | L7 (HTTP/HTTPS)                                                                         | L7 (HTTP/HTTPS)                                                                     | L7 (HTTP/HTTPS)                                                        |
| **Managed by**                 | Azure platform (external Load Balancer resource)                       | In-cluster pods (community controller)                                                  | Azure Application Gateway service (external to cluster)                             | Azure Application Gateway for Containers service (external), with in-cluster ALB controller |
| **Routing (Host/Path)**        | No (all traffic to one service)                                        | Yes (Ingress rules)                                                                     | Yes (Ingress rules -> App Gateway config)                                           | Yes (Gateway API HTTPRoutes)                                           |
| **Advanced HTTP Routing**      | No                                                                     | Limited (some via custom NGINX config or annotations)                                   | Limited (no header-based routing in AGIC)                                           | Yes (headers, methods, query strings, etc. supported)                  |
| **SSL/TLS Termination**        | No (pass-through only)                                                 | Yes (Ingress controller handles TLS; integrates with cert manager/Key Vault)            | Yes (Azure App Gateway offloads TLS)                                                | Yes (Gateway API provides TLS at Gateway or Route level)               |
| **Web Application Firewall**   | No                                                                     | No (NGINX OSS has no WAF; can use ModSecurity as addon)                                 | Yes (App Gateway WAF_v2 with managed rules)                                         | Yes (App Gateway for Containers – WAF support, currently preview)      |
| **Traffic Splitting / Canary** | No                                                                     | Basic (can deploy multiple Ingress with separate paths or use service mesh)             | No (not natively, AppGW had no weight-based routing)                                | Yes (Gateway API has weighted backends for canary, blue-green)         |
| **mTLS to Pods**               | No                                                                     | No (would require custom sidecars)                                                      | No (AppGW to pods uses TLS but not mutual auth)                                     | Yes (Mutual TLS to backend enabled)                                    |
| **Requires AKS resources**     | Minimal (just uses kube-proxy on nodes)                                | Medium (Ingress pods consume cluster CPU/RAM)                                           | Low (only a controller pod; data plane is Azure)                                    | Low (ALB controller pods, data plane on Azure)                         |
| **Scalability**                | Scales with cluster nodes (and Azure LB can handle high throughput L4) | Scale by adding controller replicas; limited by node resources for very high throughput | App Gateway autoscales independently of cluster (up to AppGW limits: \~100+ routes) | App Gateway for Containers autoscales; designed for near real-time updates and higher limits (more routes, more pods) |
| **Internal (Private) Support** | Yes (set internal annotation on Service)                               | Yes (run ingress Service as internal LB)                                                | Partially (Supported via private frontend IP or Private Link on App Gateway v2)     | Partially (Private mode in preview; otherwise use internal LB for now) |
| **Gateway API Support**        | N/A (L4 only)                                                          | Ingress only                                                                            | Ingress only (no Gateway API)                                                       | Yes (full support – designed for Gateway API)                          |

As the table shows, each option has strengths. For example, **Azure’s Application Gateway (AGIC/AGC)** solutions bring enterprise-grade security (WAF, mTLS) and advanced routing, whereas **NGINX** is straightforward and fully in your control (with a rich ecosystem of plugins/annotations). Azure’s L4 load balancer is always involved at some level (except in pure layer-7 external modes like App Gateway), but it’s transparent in most cases.

## Putting it All Together: Recommendations

Choosing an ingress and exposure strategy on AKS depends on your application needs and organizational requirements:

- **Small deployments / Basic web apps:** *Managed NGINX Ingress* is a great default. It’s easy to enable, and you get all standard Ingress functionality. For a few services or moderate traffic, NGINX is perfectly sufficient. You can terminate SSL in-cluster or even use Azure Key Vault via the Secrets Store CSI driver to supply certs. This is often the quickest path to get an app exposed on AKS with minimal fuss.

- **Enterprise apps / security sensitive:** If you require a Web Application Firewall, or want Azure to manage the ingress layer, consider **Application Gateway Ingress Controller (AGIC)**. This will give you an Azure-managed entry point with WAF, HTTPS offload, and no extra load on your AKS nodes. It’s a bit more setup (need an App Gateway and Azure permission config), but it’s worth it for production apps that need to defend against threats or handle large traffic with auto-scaling. Microsoft publishes guidance and best practices [for Azure Front Door WAF](https://learn.microsoft.com/en-us/azure/web-application-firewall/afds/afds-overview) (since App Gateway WAF is the same core as Azure Front Door WAF).

- **Advanced scenarios / future-proofing:** If you have very complex routing needs, or you want to adopt the **new Gateway API**, look at **AGC (Application Gateway for Containers)**. As of 2025, it’s the cutting-edge solution that can do things like weighted traffic splits (for canary releases) and header-based routing natively. It’s also likely the direction Azure will head in the future for fully managed ingress. Early adopters can start using Gateway API on AKS (perhaps with AGC in non-critical environments) to get familiarity, while still maybe running AGIC or NGINX for current production until AGC matures a bit more. The transition path from AGIC to AGC is designed to be smooth when you’re ready.

- **Service Mesh integration:** If you are using Istio service mesh on AKS (maybe via the Open Service Mesh add-on or the new managed Istio add-on), leveraging the **Istio Ingress Gateway** might make sense. It allows you to enforce mTLS from ingress through to your services and use Istio’s traffic policies at the edge. However, running Istio just for ingress is not necessary if you’re not otherwise using a mesh – in that case, stick to the simpler NGINX/AGIC solutions.

- **Internal-only apps:** For purely internal line-of-business applications (no public exposure), you can use any of the above in internal mode. The simplest is NGINX with an internal Load Balancer. AGIC can also do internal via Private Link as discussed. Choose based on whether you need the extra features of App Gateway. If not, a basic internal ingress on NGINX (perhaps combined with Azure Private Link to on-prem) is straightforward.

Finally, keep in mind **you can mix and match** in one cluster. Kubernetes allows multiple ingress controllers concurrently (you’d segregate them by ingress class). For instance, some teams could use NGINX ingress for one app, while another critical app uses AGIC with WAF. They will simply each watch for their respective ingress classes. Just be careful to avoid overlapping host URLs between them.

I have a follow-up post in the works about Kubernetes Gateway API and the differences between AGIC and AGC, and more about AKS and other Azure container services. Stay tuned!

-- Juanma
