---
title: "Networking"
date: 2026-03-01
weight: 30
tags: ["networking", "cilium", "metallb", "gateway-api", "tailscale", "sr-iov"]
summary: "Cilium, MetalLB, Gateway API, Tailscale, and a 200 Gbps RDMA interconnect."
---

## CNI: Cilium

[Cilium](https://cilium.io/) provides the primary CNI, installed by Talos/Omni (not managed in the GitOps repo). It runs in eBPF mode, which means network policy enforcement happens in the kernel without iptables.

One important configuration: `cni.exclusive=false`. This allows [Multus](https://github.com/k8snetworkplumbingwg/multus-cni) to attach secondary network interfaces to pods — critical for the SR-IOV RDMA setup on the DGX Sparks.

[Hubble](https://docs.cilium.io/en/stable/observability/hubble/) provides network flow observability, accessible at `hubble.wcloud.sh`.

## Load balancing: MetalLB

[MetalLB](https://metallb.universe.tf/) runs in L2 mode, advertising service IPs via ARP on the home network.

**IP pool:** `192.168.4.2` – `192.168.4.9`

A subtle challenge with MetalLB on this cluster: the DGX Sparks have two network interfaces — the 1 GbE `enP7s7` (home network) and the 200 Gbps ConnectX-7 QSFP interface. MetalLB must only advertise on the home network interface, not the QSFP link. And the interface names differ between node types (`enp12s0` on the Ryzen, `enP7s7` on the DGX Sparks).

The solution is separate `L2Advertisement` resources per node type, each specifying the correct interface name via `nodeSelectors` and `interfaces`.

## Ingress: kgateway (Gateway API)

[kgateway](https://kgateway.io/) provides the ingress layer using the Kubernetes [Gateway API](https://gateway-api.sigs.k8s.io/). It runs Envoy as the data plane.

- **External VIP:** `192.168.4.2` (from MetalLB)
- **TLS termination** with SNI-based routing
- **HTTPRoutes** define how traffic reaches each service

All services are exposed under `*.wcloud.sh`. A few examples:

| Service | URL |
|---------|-----|
| Grafana | `grafana.wcloud.sh` |
| ArgoCD | `argo-cd.wcloud.sh` |
| Keycloak | `keycloak.wcloud.sh` |
| Temporal | `temporal.wcloud.sh` |
| Longhorn | `longhorn.wcloud.sh` |

## TLS: cert-manager

[cert-manager](https://cert-manager.io/) provisions TLS certificates from Let's Encrypt using Cloudflare DNS-01 challenges. The Cloudflare API token is synced from Infisical via the External Secrets Operator.

## DNS: external-dns

[external-dns](https://github.com/kubernetes-sigs/external-dns) watches Gateway API HTTPRoute resources and automatically creates/updates CNAME records in Cloudflare. When I add a new HTTPRoute, the DNS record appears within minutes.

## Remote access: Tailscale

The [Tailscale Operator](https://tailscale.com/kb/1236/kubernetes-operator) runs a subnet router (3 replicas, one per node) that advertises:

- `192.168.4.0/24` — home network
- `10.96.0.0/12` — Kubernetes ClusterIP range
- `10.244.0.0/16` — Pod CIDR

This means from any device on my Tailnet, I can reach any service, pod, or node in the cluster directly.

## SR-IOV and Multus

The DGX Spark QSFP interconnect uses a separate networking stack for RDMA:

- **[Multus CNI](https://github.com/k8snetworkplumbingwg/multus-cni)** — attaches a secondary `net1` interface to pods alongside the primary Cilium `eth0`
- **SR-IOV Device Plugin** — creates Virtual Functions on the ConnectX-7 NICs and advertises them as `nvidia.com/cx7_qsfp` resources
- **NetworkAttachmentDefinition** — defines the `dgx-qsfp` network with IPAM on `10.100.0.0/24` and RDMA enabled

This is covered in detail on the [GPU & AI](/infrastructure/gpu/) page and in the [SR-IOV on Talos](/posts/sriov-on-talos/) journal entry.
