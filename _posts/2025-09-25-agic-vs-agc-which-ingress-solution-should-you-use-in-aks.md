---
title: "AGIC vs AGC: Which Ingress Solution Should You Use in AKS?"
date: 2025-09-25 18:15:00 Europe/Madrid
type: post
classes: wide
published: true
status: publish
categories:
- Microsoft
- Azure
tags:
- Microsoft
- Kubernetes
- Cloud-Native
- Nginx
- AKS
author: juan_manuel_rey
comments: true
---

Welcome back! This is the third and final post in my series on ingress and load balancing for Azure Kubernetes Service (AKS).

In the [first article]({% post_url 22025-09-10-exposing-aks-workloads-on-azure-2025-edition %}) we walked through the different ways you can expose workloads running in AKS. Then in the [second article]({ % post_url 2025-09-19-a-deep-dive-into-the-kubernetes-gateway-api %}) we went deep into the new Kubernetes Gateway API and what it means for AKS.

Now, to wrap things up, let’s talk about the two main Azure-native ingress options: **Application Gateway Ingress Controller (AGIC)** and the newer **Application Gateway for Containers (AGC)**. Both are Layer 7 load balancers designed to front your Kubernetes apps, but they take pretty different approaches. AGIC runs inside your cluster and configures a standard Azure Application Gateway resource based on your Ingress objects. AGC, on the other hand, is a **next-gen, cloud-managed gateway** built for Kubernetes from the ground up. It implements Azure’s flavor of the Gateway API and brings a bunch of new traffic management features.

In short, AGIC has been the go-to option for years and works well for many production setups, while AGC is the new kid on the block, aiming to solve some of AGIC’s pain points and scale better for modern workloads.

This post gives you a side-by-side look at both, their key differences, and some guidance on when to use which.

## Feature Comparison of AGIC vs. AGC

At a high level, both AGIC and AGC cover the basics: host- and path-based HTTP routing, TLS termination, and integration with Azure networking. But AGC takes things further with features that AGIC just can’t do due to how it’s built.

Here’s a quick rundown of the main differences:

| **Feature Area**             | **AGIC**                                      | **AGC**                                         |
|------------------------------|-----------------------------------------------|-------------------------------------------------|
| **Kubernetes API Support**   | Ingress API (networking.k8s.io/v1) only       | Ingress API and Gateway API (GA, v1.3+)         |
| **Advanced Routing**         | Host, path-based; limited header/query/method | Host/path, plus header, query, method-based     |
| **Traffic Management**       | No built-in split/weighted routing            | Weighted traffic split, canary, mTLS, retries   |
| **Architecture**             | External App Gateway, controlled by AGIC pod  | Dedicated Azure resource + ALB controller in AKS|
| **Deployment Modes**         | Helm, AKS add-on                              | Managed (by ALB controller) or BYO ARM resource |
| **Autoscale Support**        | App Gateway V2/WAFV2 with autoscale           | Designed for seamless scalability, higher limits |
| **Maximum Listeners (V2)**   | 100 listeners (hard limit, cannot increase)   | Expected higher, scales with Gateway API model   |
| **Update Latency**           | May lag (resource update delays common)       | Near real-time config updates                   |
| **Security**                 | WAFv2, End-to-End TLS, session affinity       | WAF (preview), mTLS, modern policy model        |
| **Private IP Ingress**       | Supported                                     | Not yet (GA soon), only public FQDN (as of 09/25)|
| **Pricing Model**            | Instance (V2), capacity units, data transfer  | Per resource (AGC/Frontend/Assoc), capacity unit|
| **Kubernetes Networking**    | Azure CNI, Kubenet, CNI Overlay now GA        | Azure CNI, Azure CNI Overlay (GA)               |
| **Migration**                | Mature, widely used, helm/add-on options      | Incremental migration path from AGIC            |
| **Community Support**        | Broad, mature, rich best practices            | Active, rapidly evolving, limited production use|
| **GA Status**                | Generally Available                           | In late preview/GA imminent (as of Sept. 2025)  |

## Architectural Differences

### AGIC Core Components and Architecture

Azure Application Gateway Ingress Controller (AGIC) is a Kubernetes controller that runs as a pod inside your AKS cluster. Its purpose is to provide seamless integration between native Kubernetes ingress resources and Azure Application Gateway, an Azure-managed Layer 7 (HTTP/HTTPS) load balancer.

Here are the core architectural aspects:

- **AGIC Pod**: Monitors AKS for Ingress resources and reflects desired configuration to the Azure Application Gateway via Azure Resource Manager (ARM) APIs.
- **Application Gateway V2/WAF V2**: Acts as the actual traffic entry point; managed via ARM.
- **Resource Ownership**: Application Gateway is an independent Azure resource, (can be shared or exclusively controlled by AGIC).
- **Connection Model**: AGIC binds Application Gateway directly to Kubernetes Pods’ private IPs, bypassing Service and KubeProxy hops.
- **Deployment Modes**:
  - *Helm Chart*: Flexible, customizable, supports advanced features (e.g., multiple namespaces, prohibited targets).
  - *AKS Add-on*: One-line deployment, managed updates, but with some configuration limitations (e.g., no ProhibitedTargets).

AGIC works with Azure CNI, CNI Overlay, and even Kubenet. Thanks to recent updates, Overlay networking is fully supported, which helps a lot for larger, multi-tenant clusters.

**Implications**: AGIC makes it easy to use Azure Application Gateway as the ingress for AKS without running multiple proxies or extra load balancers. However, each cluster’s update to AGIC-Application Gateway configuration is serialized through ARM, which can introduce lag and occasionally causes reconciliation delays (see performance discussion below).

### AGC Core Components and Architecture

Application Gateway for Containers (AGC) is the next-generation Azure-native ingress and Layer 7 load balancing solution, purpose-built for Kubernetes and deeply aligned with the Kubernetes Gateway API:

- **AGC Azure Resource**: AGC is not traditional Application Gateway—it is its own distinct resource class, surfaced and managed independently.
- **Control Plane**: An Azure-managed control plane orchestrates the configuration and provisioning of proxy components (data plane).
- **Frontend Resource**: Each AGC instance can have multiple frontends (up to 5 as of 09/25), each with a unique FQDN (no private IP yet—see roadmap).
- **Association Resource**: Each frontend is associated with a subnet in your VNet (must be delegated for AGC use, requires at least /24 prefix, 256 IPs per frontend).
- **ALB Controller**: Deployed as a set of Kubernetes pods, it watches for Kubernetes-native Gateway API and Ingress resources, translates them, and orchestrates AGC configuration via Azure ARM API.
- **Managed Deployment**: AGC resources can be managed entirely by ALB controller (zero-touch from IaC or Portal), or provisioned manually for more separation and control ("Bring Your Own Deployment").

Because AGC is both cloud-native (lives as an Azure resource, with Azure identity/workload identity support) and Kubernetes-native, it’s faster and more flexible than AGIC. Config updates usually hit within seconds, and the Gateway API gives you way more routing options.

**Implications**: AGC is more tightly bound to Kubernetes conventions, enables advanced L7 routing via the Gateway API, and is designed for near-real-time updates and higher scale as compared to the legacy ARM/AGIC model.

## Use Cases

### AGIC Use Cases

AGIC is best suited for scenarios where:

- You need an Azure-managed, production-ready ingress controller, tightly integrated with Azure’s Application Gateway L7 load balancer.
- Your workloads require features such as:
  - Path-based and host-based routing
  - Cookie-based session affinity
  - Public/private/hybrid site exposure
  - End-to-end TLS (frontend termination and backend re-encryption)
  - Integrated Web Application Firewall (WAF) following OWASP Core Rule Set (CRS)
  - Multi-namespace support for multi-tenant AKS clusters
- You need compatibility with existing Ingress manifest patterns, as used with NGINX, Traefik, etc., but want Azure-native provisioning, security (WAF), and autoscaling.

Typical deployment patterns include:

- **Cluster-level public ingress** for internet-facing AKS services.
- Ingress with **WAF** to centrally protect multiple applications behind a unified set of security policies.
- Multi-tenancy, where each tenant is isolated by Ingress resource plus Application Gateway listeners/hosts.

Bottom line, AGIC is mature, well-documented, and fits regulated environments or anyone running multi-environment rollouts (dev/test/prod) today.

### AGC Use Cases

AGC addresses next-generation ingress needs for cloud-native, advanced, and high-scale applications:

- **Advanced Routing**:
  - Header-based, query-based, and method-based routing
  - Weighted routing for canary and blue/green deployments
  - mTLS for enhanced security between gateway and backend pods
  - Robust traffic splitting and automatic retry logic
- **Modern API Surface**:
  - Native support for the Kubernetes Gateway API, enabling advanced role separation between platform and application teams, portable route manifests, and rich policy models
- **Rapid Update & Scale**:
  - Designed for near-real-time sync between Kubernetes manifests and Azure load balancing infrastructure, minimizing downtime and increasing deployment speed during pod rollouts, scaling, and blue/green deployment events
- **Zero-touch Management**:
  - Optional mode where AGC and its supporting Azure resources (frontends, associations, security policies) can be provisioned, scaled, and managed entirely through Kubernetes CRDs, with no need for direct ARM or portal intervention

Ideal scenarios include:

- Teams adopting the Gateway API for future-proofing cluster ingress
- Workloads requiring complex, dynamic routing at L7
- High-scale AKS clusters exceeding AGIC’s listener and pod count limitations
- Environments where split traffic, advanced mTLS, and flexible update policies are required natively from the ingress controller.

**Note**: As of September 2025, AGC is quickly maturing and becoming GA in most Azure regions, but it may not yet be the default ingress for all enterprise scenarios, particularly those that require private frontend IPs. It is, however, the clear future direction for Azure-native Kubernetes ingress.

## Performance Characteristics

### AGIC Performance

**Strengths**:

- AGIC eliminates extra network hops compared to using an in-cluster ingress controller plus external load balancer, as Application Gateway directly routes to pod IPs.
- Uses Application Gateway autoscale (V2 SKUs) for handling increased L7 traffic.

**Limitations**:

- Real-world users have documented occasions where Application Gateway backend pool updates can take from several minutes up to 45 minutes, especially when performing large deployments, application upgrades, or frequent pod churn. During this time, new pods may not receive traffic, causing temporary outages or degraded service.
- The update speed is limited by the ARM API and rate limits, as well as the Cloud platform’s underlying propagation of configuration changes within Application Gateway.

AGIC suits workloads where minute-level update lag can be tolerated, but is less suitable for extremely dynamic environments where zero downtime and rapid rollout of changes are critical.

### AGC Performance

**Strengths**:

- AGC delivers *near-real-time* updates to backend pools, listener configuration, and routing rules, greatly reducing update lag documented in AGIC deployments.
- With tight integration to the Gateway API and custom resource reconciliation, the ALB controller makes updates as soon as new pods/routes are registered, typically within seconds.

**Advanced Features**:

- Real-world user tests show updates to routing rules in AGC propagate significantly faster than with AGIC—boosting reliability and making advanced patterns such as traffic splitting, canary, and rapid blue/green deployment practical.
- AGC can also scale to much larger pod/backend pool and listener counts, aligning to the needs of truly high-scale, cloud-native AKS clusters.

**Limitations**:

- As of GA preview, private endpoints are not yet supported, and current deployments focus on public ingress.
- Production usage should be carefully validated, as some advanced features (such as WAF integration and all custom probe options) may be in private preview or just entering GA.

AGC is superior for high-agility environments, modern DevOps deployment pipelines, and teams requiring “cloud-native speed” in their ingress and traffic management.

## Kubernetes Integration and API Support

### AGIC: Ingress API + Azure Enhancements

- **API Surface**: AGIC supports only the Kubernetes Ingress API (`networking.k8s.io/v1`).
- **Azure Features via Annotations**: Its deeper Azure features (e.g., WAF per path, affinity, SSL redirects, custom health probes) require proprietary Kubernetes annotations.
- **CRD Extensions**: Some scenarios (host/path blocklists via ProhibitedTargets, shared gateway) are achieved with custom resource definitions (CRDs), but not standardized across upstream ingress controllers.
- **Networking Support**: As of mid-2025, AGIC supports Azure CNI, Azure CNI Overlay (since v1.9.0+), and Kubenet. Overlay support brings AGIC to very large-scale, VNet-efficient clusters.

**Deployment Modes**:

- Deploy via Helm chart (most flexible) or as AKS add-on (simplest). Add-on is limited to one Application Gateway per AKS cluster, whereas Helm can operate with more advanced multi-AGIC-on-one-AKS or multi-AKS-per-AppGW hybrid scenarios.

In summary, AGIC is tightly bound to established Ingress resource patterns, enhanced with Azure-specific extensions, but limited by the expressiveness and constraints of the original Ingress API model.

### AGC: Gateway API and Ingress API

- **API Support**: AGC is built to natively support both Kubernetes Ingress API and the **Kubernetes Gateway API** (GA v1.3.0 as of April 2025).
- **Gateway API Advantages**:
  - Rich routing: host, path, header, query, HTTP method matching natively
  - Weighted and split traffic handling (for canary, blue/green, staged rollout)
  - Role separation: clear division between platform operators (manage Gateways) and application teams (manage Routes)
  - Standardization: avoids undifferentiated controller-specific annotations, enabling portable configurations across cloud and on-prem clusters
- **CRD Support**: As the Gateway API project evolves, AGC's ALB controller tracks the GA and Beta/Extended types, supporting HTTPRoute, GRPCRoute, TCPRoute, UDPRoute, and advanced policy resources (BackendTLSPolicy, HealthCheckPolicy, etc.).

**Deployment Modes**:

- Helm-deployed ALB controller manages AGC resources based on native Gateway API CRDs.
- Full BYO mode for customers desiring separation of duty between resource provisioning and cluster configuration.

AGC clearly aligns with the Kubernetes future—emphasizing standards, portability, extensibility, and cloud-native role separation.

## Scalability and Autoscaling

### AGIC Scalability

- **Dependant on Application Gateway V2/WAFv2 Limitations**: AGIC inherits the scaling properties of the underlying App Gateway V2. Some of the key quotas:
  - **Listeners**: 100 per App Gateway V2 (hard limit, not increaseable). Hitting this cap blocks further ingress creation, requiring manual architectural changes, such as collapsing listeners via wildcards or deploying additional Gateways.
  - **Pods/Backends**: Pod IPs directly in the backend pool. In large clusters with high churn, Application Gateway resource updates can become a bottleneck, increasing reconciliation lag and possible configuration drift.
  - **Autoscaling**: Application Gateway V2 can autoscale instance count, but is still limited by capacity units and various max/min quotas.
- **Pod Autoscaling**: AGIC observes backend target health and counts. When linking with Kubernetes autoscaler and Azure Metrics Adapter, can scale backend pods in response to live traffic, based on per-backend metrics.

AGIC is suitable for mid-size clusters, but organizations building very high-scale or multi-tenant AKS may face hard scalability ceilings, especially when each ingress demands a distinct listener.

### AGC Scalability

- **Gateway API Model**: AGC is designed to scale beyond AGIC’s hard listener and resource limits, thanks to the Gateway API's object model, which decouples frontend, listener, and route configuration.
- **Backend Pool Size**: AGC supports much greater scale, expecting to scale to tens of thousands of backend endpoints once GA, with no rigid resource bottleneck on listeners or subnets.
- **Regional Availability**: As of September 2025, AGC is available in multiple regions, including Australia East, Central US, East Asia, North Europe, UK South, West Europe, and more. Wider rollout is ongoing, with new GA regions being added rapidly.
- **Deployment Patterns**: Supports classic scaling patterns but now adds first-class support for traffic splitting, zero-downtime deployments, advanced retries, and weighted routing, out-of-the-box.

Here AGC is the clear scalability winner, both in terms of limits and operability, especially for large, fast-moving AKS estates.

## Security Features and Policy Models

### AGIC Security

- **Front-End TLS**: Supports TLS termination at gateway (SSL certificate installed at the App Gateway listener), as well as End-to-End TLS which re-encrypts flow to the backend pods.
- **TLS Policy/Profiles**: Frontend TLS policy and Cipher suite can be controlled via custom annotations.
- **Backend Trust**: App Gateway v2 supports root cert trust for backends, enabling communication only with authenticated backend pods/apps.
- **Web Application Firewall (WAF)**: Full integration with WAF v2 tier, with standardized rules (OWASP 3.x CRS), custom rule support, path-level policies, and more. Policy can be set at gateway, listener, or URI path level.
- **Session Affinity**: Cookie-based sticky sessions per path/listener as needed.
- **Private Link**: Supports private ingress via Application Gateway’s private IP in VNET.

AGIC offers enterprise-grade L7 security, with mature WAF integration, TLS flexibility, multi-tenant policy, and deep conformance to enterprise security models.

### AGC Security

- **Front-End TLS**: Full support for termination, plus new model for referencing certificates in AKS Secret Store (no longer tied to Azure Key Vault).
- **End-to-End Encryption**: Secure traffic from client to Gateway and re-encrypt to backend pods (mTLS supported, with custom CA chains via BackendTLSPolicy).
- **TLS Policy**: Custom TLS Policy for frontends; note that predefined policy names and cipher suites differ from AGIC and require Gateway API definitions.
- **mTLS**: Native mutual TLS support to backends via Gateway API CRDs.
- **Web Application Firewall**: WAF support now entering General Availability, model is a one-to-many mapping via Security Policy resource.
- **Advanced Policy Model**: Policy as code—define session affinity, health checks, header rewrites, traffic splitting, canary patterns all via open, Kubernetes-standard custom resources; avoids annotation overload.

**Limitations** (as of 09/25):

- Private frontends (private IP ingress) are not yet GA.
- Some AGIC WAF features (listener-level custom rule granularity) may not be fully mapped until later in AGC’s roadmap.

AGC’s security capabilities are strong and evolving, built for modern “policy-as-code” approaches, and ready for advanced zero-trust and security-conscious workloads.

## Migration and Transition Guidance

### Migration from AGIC to AGC

The AKS team provides an [official migration guide](https://learn.microsoft.com/en-us/azure/application-gateway/for-containers/migrate-from-agic-to-agc):

- **Incremental Migration**: You can operate AGIC and AGC in parallel within the same cluster, testing and validating AGC prior to cut-over.
- **Feature Parity/Mappings**: Microsoft provides a table mapping AGIC-specific annotations to their AGC equivalents (e.g., session affinity, WAF, health probes, URL rewrites, etc.), or notes those features which are not yet supported in AGC.
- **Annotations and CRDs**: AGC migrates away from opaque ingress annotations to open, composable Gateway API CRDs.

**Known Gaps as of Sept 2025**:

- Private endpoints (private IP frontends).
- Custom frontend ports other than 80/443.
- Configurable request timeout.

Workloads depending on these features should delay migration until such dependencies are unblocked. Otherwise, Microsoft recommends starting the transition now, to take advantage of performance, scalability, and operational improvements.

## Which One Should You Choose?

Choosing between AGIC and AGC in late 2025 is largely a matter of present needs versus future alignment:

- **Choose AGIC if**:
  - You require mature, stable, fully GA-backed ingress.
  - You depend on features not yet landed in AGC: private ingress, custom ports, or certified compliance (in your region).
  - Your workloads fit comfortably beneath the listener/backend pool limits, and listener churn (update lag) is not a blocker.
  - You want a proven, “set and forget” experience suitable for current production and regulatory requirements.

- **Choose AGC if**:
  - You are designing for the future, need cloud-native ingress at scale, or are already using Gateway API constructs.
  - You require advanced L7 traffic management, canary deploys, mTLS, or policy-as-code routing and security models.
  - You want the fastest possible update velocity and are prepared for limited friction in the transition/migration journey.
  - Your use cases are rapidly scaling, multi-tenant, and you demand cost-efficient scaling beyond AGIC’s resource limits.

**Future Direction**: As the Kubernetes community and ecosystem continue to embrace Gateway API as the standard for cloud-native ingress, AGC is the clear evolutionary path forward for AKS, offering a feature-rich, highly scalable, and operationally efficient alternative to AGIC’s legacy ingress model.

If starting new projects, experiment and develop proficiency with AGC and the Gateway API in dev/test environments now. If running on AGIC in production, begin planning and validating migration, especially as AGC’s key missing features (private ingress, granular timeouts, etc.) reach GA in the months ahead. The incremental migration path is supported, enabling teams to modernize cluster ingress and traffic policies without abrupt cutovers or downtime risks.

---

With this, I wrap up our short series on ingress and load balancing for Azure Kubernetes Service. I hope you found it helpful, and as always, your comments and feedback are welcome.. Stay tuned for more articles about AKS and Kubernetes in general.

-- Juanma
