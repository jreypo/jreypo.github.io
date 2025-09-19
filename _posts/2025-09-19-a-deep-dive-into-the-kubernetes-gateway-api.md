---
title: A Deep Dive into the Kubernetes Gateway API
date: 2025-09-19 19:00:00 Europe/Madrid
type: post
classes: wide
published: true
status: publish
categories:
- Cloud-Native
tags:
- DevOps
- Kubernetes
- Cloud-Native
- Ingress
author: juan_manuel_rey
comments: true
---

In a previous article about [exposing services in Azure Kubernetes Services]({% post_url 2025-09-10-exposing-aks-workloads-on-azure-2025-edition %}) I briefly touched on the Gateway API topic. This article is a first follow up on that one, it provides a deep dive into the Gateway API: what it is, why it exists, how it works, and what it enables for Kubernetes users. In the next one I will go deeper in the differences between the two main Ingress solutions in Azure, Azure Application Gateway Ingress Controller (AGIC) and Application Gateway for Containers (AGC).

Ingress has long been the standard way to expose services in Kubernetes. While it works for many use cases, the **Ingress API** shows its limitations in modern cloud-native environments: limited routing rules, inconsistent annotations across ingress controllers, and a lack of role separation between platform and application teams. To address these issues, the Kubernetes community introduced the **Gateway API** — a next-generation, extensible API for service networking. It standardizes how traffic enters and flows through a cluster while offering flexibility and advanced features for evolving workloads.

## Why Gateway API? The Limitations of Ingress

The **Ingress API** (introduced in Kubernetes 1.2) was a significant step in standardizing traffic routing into clusters. However, over time several limitations emerged:

* **Annotations for advanced features:** Controllers like NGINX, Traefik, or cloud load balancers all extended Ingress with custom annotations. This led to fragmentation and non-portable configurations.
* **Single resource type:** Ingress objects had to define both infrastructure (load balancer) and application-level routes in one place, creating operational friction.
* **Limited expressiveness:** Ingress supported only host- and path-based routing, making features like header-based routing, canary deployments, or retries impossible without custom hacks.
* **No role separation:** Platform teams and app developers had to share the same object, with no clear boundaries for responsibility.

The Gateway API was designed to solve these problems.

## Core Concepts of the Gateway API

The Gateway API introduces a **set of Custom Resource Definitions (CRDs)** that together describe networking in a Kubernetes-native, extensible way. The key CRDs are:

1. **GatewayClass**

   * Defines a type of Gateway provided by an implementation (e.g., NGINX, Istio, Azure AGC).
   * Similar to how `StorageClass` defines different storage backends.
   * Example: `azure-application-gateway`, `nginx-gateway`.

2. **Gateway**

   * An instance of a GatewayClass.
   * Represents a data-plane resource (like a load balancer or proxy) that handles traffic.
   * Can be shared across namespaces or dedicated to one.

3. **Routes** (e.g., **HTTPRoute**, **TCPRoute**, **UDPRoute**, **GRPCRoute**)

   * Define how traffic is routed to backends.
   * HTTPRoute is the most commonly used today.
   * Decouples routing logic from the infrastructure (Gateway).

4. **BackendPolicy / ReferencePolicy**

   * Fine-grained configuration for backends and cross-namespace references.

### Example: HTTPRoute

Here’s a simple Gateway + HTTPRoute definition:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
  namespace: default
spec:
  gatewayClassName: azure-application-gateway
  listeners:
    - name: http
      protocol: HTTP
      port: 80
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-routes
  namespace: default
spec:
  parentRefs:
    - name: web-gateway
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /app
      backendRefs:
        - name: app-service
          port: 80
```

This configuration creates:

* A Gateway (`web-gateway`) using Azure’s Application Gateway implementation.
* An HTTPRoute (`web-routes`) that sends traffic for `/app` to `app-service`.

## Advantages of Gateway API

1. **Standardization Across Implementations**

   * No more controller-specific annotations. Features like header-based routing, traffic splitting, and retries are first-class in the API.

2. **Separation of Concerns**

   * Platform teams manage **Gateways** (infrastructure).
   * App teams manage **Routes** (application traffic).
   * Clear ownership and security boundaries.

3. **Advanced Routing**

   * Beyond host/path: route by headers, query params, methods, or weight (for canary/blue-green).

4. **Multi-protocol Support**

   * Not just HTTP/HTTPS, but also **TCP, UDP, gRPC** via dedicated Route types.

5. **Portability**

   * Works across different environments (cloud load balancers, service meshes, open-source ingress controllers).

6. **Extensibility**

   * New route types and policy resources can be added without breaking the spec.

## Gateway API in the Kubernetes Ecosystem

* **Service Mesh Integration:** Service meshes like Istio and Linkerd are adopting Gateway API to unify ingress and mesh routing.
* **Cloud Providers:** Azure (AGC), Google Cloud (GCLB), and AWS are all building Gateway API support into their managed load balancers.
* **Ingress Controllers:** Projects like NGINX Gateway, Contour, and Traefik support Gateway API natively.

This makes Gateway API a **unifying standard** across diverse networking implementations.

## Gateway API vs. Ingress: Quick Comparison

| Feature                           | Ingress API       | Gateway API                       |
| --------------------------------- | ----------------- | --------------------------------- |
| Routing options                   | Host/Path         | Host, Path, Header, Query, Method |
| Multi-protocol support            | HTTP only         | HTTP, TCP, UDP, gRPC              |
| Role separation (platform vs dev) | No                | Yes                               |
| Extensibility                     | Limited           | High                              |
| Standardization across vendors    | Low (annotations) | High                              |
| Expressiveness for traffic mgmt   | Low               | High                              |

## Current Status of Gateway API (as of K8s v1.34)

1. **Gateway API GA resources**
   The following resources in `gateway.networking.k8s.io/v1` are **stable (GA)** and fully supported:

   * `GatewayClass` — defines classes of gateways and controllers that implement them.
   * `Gateway` — instance resource specifying listeners, addresses, protocol, etc.
   * `HTTPRoute` — for HTTP/HTTPS routing rules (host, path, headers, query params, etc.).
   * `GRPCRoute` — for mapping gRPC traffic.

2. **Beta/Alpha or Extended Resources**
   There are additional route types and policy resources beyond these GA types, which are in **beta**, **alpha**, or **“extended”** support. Some of them:

   * `TCPRoute`, `UDPRoute` — for TCP/UDP protocol-based routing. These are generally not GA universally; some implementations support them (depending on the version of the implementation and its conformance level).
   * `TLSRoute` — matching TLS-specific metadata (SNI etc.). Also in more experimental/extended support in many implementations.
   * `BackendTLSPolicy` — for specifying TLS behavior to backends. Some parts in Core, some in Extended. For example, changes in `SubjectAltNames` field moved from core support to extended in Gateway API v1.3.0

3. **Gateway API Version / Spec release**

   * The project is under **SIG Network / Kubernetes-SIG** and the spec is versioned separately. The latest released spec version is **v1.3.0** (released April 2025).
   * v1.3.0 includes enhanced TLS handling (e.g., `OverlappingTLSConfig`, clarified behavior for SNI/hostname matching) and conformance test improvements.

4. **Kubernetes v1.34 and Gateway API**

   * Kubernetes **v1.34** (released \~August 2025) does not deprecate any of the GA Gateway API resources.
   * Since Gateway API is implemented via CRDs (custom resources) and controllers, support depends on whether the chosen ingress/gateway controller implementation supports the GA API and the extended resources you need. Kubernetes being at v1.34 means the cluster supports CRD installations; but you still need your controller to support the version of spec you need (e.g. v1 or v1beta / alpha for non-GA types).

5. **Implementation Landscape & Conformance**

   * There are multiple implementations (cloud-native, open-source) that support at least the core GA types of Gateway API (`GatewayClass`, `Gateway`, `HTTPRoute`, `GRPCRoute`).
   * Some implementations have partial support for extended or non-GA route types (TCPRoute, UDPRoute, TLSRoute), with varying levels of stability.
   * The Gateway API project tracks **conformance reports**, which show which controllers are “conformant” vs. “partially conformant”. If a controller claims to fully support HTTPRoute, for example, it should pass all core conformance tests for the HTTP profile.

## What You Should Check Before Adopting Gateway API in Your Cluster

Since Kubernetes supports it via CRDs, the main dependency is on your **gateway controller** (implementation). Here are key things to verify:

1. **Spec Version Support**
   Does your controller support Gateway API v1.x (at least v1.2 or v1.3)? If you need features in extended/alpha types (e.g. TLSRoute, UDPRoute), is that implemented?

2. **Routing Features**
   If your workload requires header-based routing, query param matching, traffic splitting, etc., check whether those are GA or still experimental/extended.

3. **TLS Handling / SNI Behavior**
   Clarify behavior when multiple hostnames or wildcard hosts are involved, and how the controller behaves when TLS listener + HTTPRoute hostnames overlap.

4. **Controller Conformance**
   Review the conformance reports for the controller. Does it support the core profile you need? Are there known gaps?

5. **Fallback or Compatibility**
   If migrating from Ingress, ensure you have fallback or mixed support while you transition. Some controllers allow both Ingress and Gateway API resources in parallel.

6. **Performance & Scaling**
   Although the GA resources are stable, each controller may have its own performance limits. Check how many listeners/routes your controller can handle, how fast it propagates config changes, etc.

7. **Security & Policies**
   Does the controller support capabilities you need: TLS termination, mutual TLS, policy for backend TLS, reference grants (for cross-namespace references), etc.

---

The **Gateway API** represents the future of Kubernetes ingress and service networking. It solves long-standing pain points of the Ingress API, introduces powerful traffic management capabilities, and establishes a vendor-neutral standard across clouds and open-source controllers.

For Azure users, this API is already making its way into services like Azure **Application Gateway for Containers (AGC)** — meaning the sooner you adopt Gateway API, the more aligned your workloads will be with Kubernetes’ networking future.

See you in the next article where we will explore the differences between **Azure Application Gateway Ingress Controller (AGIC**) and **Application Gateway for Containers (AGC)**.

--Juanma
