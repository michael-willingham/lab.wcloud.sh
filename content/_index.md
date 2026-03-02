---
title: "Willingham Cloud"
description: "A homelab Kubernetes cluster built for learning GPU infrastructure, RDMA networking, and platform engineering."
---

A 3-node Kubernetes cluster running on [Talos Linux](https://www.talos.dev/), managed entirely through GitOps with [FluxCD](https://fluxcd.io/).

> **Note:**
This site is a living document. I use it to track what I'm building, what I'm learning, and what I still don't understand.

## 🖥️ The hardware

| Node | Hardware | CPU | RAM | Role |
|------|----------|-----|-----|------|
| `talos-76w-3r0` | Custom PC | AMD Ryzen 9 9950X3D (16c/32t) | 192 GB DDR5 | Control plane + workloads + GPU |
| `talos-7aj-lwl` | NVIDIA DGX Spark | GB10 Grace Blackwell (20c ARM) | 128 GB | Control plane + GPU workloads |
| `talos-ysi-4k0` | NVIDIA DGX Spark | GB10 Grace Blackwell (20c ARM) | 128 GB | Control plane + GPU workloads |

**Totals:** 56 cores, 448 GB RAM, ~3.4 PFLOPS FP4 AI compute.

The two DGX Sparks are connected via a **200 Gbps QSFP DAC** direct-attach cable using Mellanox ConnectX-7 NICs with SR-IOV, enabling GPU-Direct RDMA for multi-node training with [NCCL](https://developer.nvidia.com/nccl).

## 🚀 What's running

The cluster runs a full platform stack — not because everything is needed, but because each component is something I wanted to understand deeply.

**GitOps:** FluxCD reconciles everything from Git. No `kubectl apply`, ever.

**Networking:** Cilium (eBPF CNI), MetalLB (L2), kgateway (Gateway API / Envoy), Tailscale subnet routing

**Storage:** Longhorn (v1 data engine)

**Identity:** Keycloak providing OIDC SSO for every web UI in the cluster

**Observability:** VictoriaMetrics, Grafana, Jaeger (OpenTelemetry), Hubble

**Messaging:** Kafka (Strimzi, KRaft mode), NATS (JetStream), RabbitMQ

**Serverless:** Knative Serving + Eventing (backed by Kafka)

**Dev Platform:** Coder, Temporal, GitHub Actions runners (ARC), HCP Terraform agents, Camel K

**GPU/AI:** NVIDIA device plugin, SR-IOV device plugin, Multus CNI for RDMA interfaces


## 🧠 What I'm learning right now

My current focus is on **RDMA and SR-IOV** — figuring out how to get multi-node GPU training working efficiently across the two DGX Sparks using the 200 Gbps ConnectX-7 interconnect. It's been a deep rabbit hole involving:

SR-IOV Virtual Functions on Talos Linux (which has a read-only root filesystem)

Multus CNI for secondary network interfaces

NCCL configuration for GPU-Direct RDMA

The gap between "it works in a single node" and "it works across nodes"

Read more in the [journal](/posts/).

## 🏗️ Design philosophy

A few deliberate choices:

1. **All nodes are control-plane.** Three nodes, all running both control plane and workloads. Typical for a homelab, but it means every component needs to tolerate this.
2. **Mixed architecture.** AMD64 (Ryzen) and ARM64 (DGX Spark). CI runners and Terraform agents have separate scale sets for each arch.
3. **GitOps only.** Every change goes through Git. FluxCD reconciles. If it's not in the repo, it doesn't exist.
