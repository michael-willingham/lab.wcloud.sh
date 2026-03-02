---
title: "Architecture"
date: 2026-03-01
weight: 20
tags: ["architecture", "fluxcd", "gitops"]
summary: "How the cluster is organized — FluxCD dependency chains, traffic flow, and component relationships."
---

## FluxCD dependency chain

Everything in the cluster is deployed via FluxCD Kustomizations, organized in a dependency chain that ensures components are created in the right order:

{{< mermaid >}}
graph TD
    NS["namespaces"] --> CRDs["crds"]
    CRDs --> Routes["routes"]
    CRDs --> Base["base<br/><small>cert-manager, ESO, MetalLB,<br/>Longhorn, kgateway</small>"]
    Base --> Operators["operators<br/><small>Strimzi, OLM, Percona,<br/>KEDA, Dragonfly, etc.</small>"]
    Operators --> Network["network<br/><small>MetalLB pools, L2Adv,<br/>Gateway, HTTPRoutes</small>"]
    Operators --> Messaging["messaging<br/><small>Kafka, NATS, RabbitMQ</small>"]
    Operators --> DevPlatform["dev-platform<br/><small>Coder, ARC, TFC agents,<br/>Camel K</small>"]
    Operators --> Observability["observability<br/><small>VMKS, Jaeger,<br/>Headlamp</small>"]
    Operators --> Security["security<br/><small>Keycloak</small>"]
    Operators --> Argo["argo<br/><small>ArgoCD, Workflows,<br/>Events</small>"]
    Observability --> VPA["vpa<br/><small>VPA resources</small>"]
{{< /mermaid >}}

Each box is a FluxCD Kustomization pointing to a directory in the [Git repository](https://github.com/willingham-cloud/k8s-config). Arrows represent `dependsOn` relationships — a Kustomization won't reconcile until its dependencies are healthy.

### Why this ordering matters

- **Namespaces first** — every other resource needs its namespace to exist.
- **CRDs before base** — operators like cert-manager need their CRDs registered before the HelmRelease can create CRs.
- **Base before operators** — cert-manager needs to be running before anything that needs TLS certificates. ESO needs to be running before anything that needs secrets from Infisical.
- **Security and Argo deploy in parallel** — ArgoCD configures Keycloak OIDC endpoints, but doesn't need Keycloak running to deploy. Login works once Keycloak comes up naturally.

## Traffic flow

External traffic enters the cluster through a single path:

{{< mermaid >}}
graph LR
    Client["Client<br/>(on Tailnet)"] --> DNS["Cloudflare DNS<br/>*.wcloud.sh"]
    DNS --> TS["Tailscale<br/>Subnet Router"]
    TS --> VIP["MetalLB VIP<br/>192.168.4.2"]
    VIP --> GW["kgateway<br/>(Envoy)"]
    GW -->|"SNI routing"| Svc["Kubernetes<br/>Service"]
    Svc --> Pod["Pod"]

    CM["cert-manager"] -.->|"TLS certs"| GW
    ExtDNS["external-dns"] -.->|"DNS records"| DNS
{{< /mermaid >}}

Key points:

- All services are exposed under `*.wcloud.sh` and are only reachable via Tailscale.
- **external-dns** watches Gateway API HTTPRoute resources and automatically creates Cloudflare DNS records.
- **cert-manager** provisions Let's Encrypt TLS certificates using Cloudflare DNS-01 challenges.
- **kgateway** (Envoy-based) performs TLS termination and SNI-based routing.
- **MetalLB** advertises the VIP (`192.168.4.2`) via L2/ARP on the home network.

## Repository structure

```
k8s-config/
├── clusters/willingham-k8s/   # FluxCD Kustomization entry points
├── namespaces/                # Namespace definitions
├── crds/                      # Custom Resource Definitions
├── base/                      # Foundational operators
│   ├── cert-manager/
│   ├── external-secrets/
│   ├── longhorn/
│   ├── metallb/
│   └── kgateway/
├── operators/                 # Infrastructure operators
├── network/                   # MetalLB pools, Gateway, routes
├── messaging/                 # Kafka, NATS, RabbitMQ
├── security/                  # Keycloak
├── observability/             # VMKS, Jaeger, Headlamp
├── argo/                      # ArgoCD, Workflows, Events
├── dev-platform/              # Coder, ARC, TFC, Camel K
├── vpa/                       # VPA resources
├── routes/                    # Gateway API HTTPRoutes
└── docs/                      # Documentation
```

Every directory corresponds to a FluxCD Kustomization. The cluster reconciles the `master` branch continuously.
