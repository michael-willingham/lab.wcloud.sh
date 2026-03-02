---
title: "Stack Reference"
date: 2026-03-01
weight: 100
tags: ["reference"]
summary: "Every component running in the cluster, at a glance."
---

A complete reference of every component running in the cluster.

## Core platform

| Component | Namespace | Notes |
|-----------|-----------|-------|
| Talos Linux | — | Immutable OS, managed via Omni |
| Kubernetes | — | containerd runtime |
| FluxCD | `flux-system` | helm-controller, kustomize-controller, source-controller, notification-controller |
| Cilium | `kube-system` | eBPF CNI, installed by Talos/Omni |

## Networking

| Component | Namespace | Notes |
|-----------|-----------|-------|
| MetalLB | `metallb-system` | L2 mode, pool `192.168.4.2-9` |
| kgateway | `kgateway` | Gateway API / Envoy |
| cert-manager | `cert-manager` | Let's Encrypt, Cloudflare DNS-01 |
| external-dns | `external-dns` | Cloudflare provider |
| Tailscale Operator | `tailscale` | Subnet router, 3 replicas |
| Multus CNI | `kube-system` | Thick plugin, secondary interfaces |

## GPU & SR-IOV

| Component | Namespace | Notes |
|-----------|-----------|-------|
| NVIDIA Device Plugin | `kube-system` | GPU resource advertising |
| SR-IOV Device Plugin | `kube-system` | Standalone (no operator), 4 VFs/node |

## Storage

| Component | Namespace | Notes |
|-----------|-----------|-------|
| Longhorn | `longhorn-system` | v2 data engine (SPDK) |

## Identity & secrets

| Component | Namespace | Notes |
|-----------|-----------|-------|
| Keycloak | `keycloak` | `wcloud` realm, OIDC SSO |
| External Secrets Operator | `external-secrets` | Infisical Cloud backend |

## Observability

| Component | Namespace | Notes |
|-----------|-----------|-------|
| VictoriaMetrics K8s Stack | `vmks` | Metrics + Grafana + AlertManager |
| OpenTelemetry Operator | `otel-system` | Manages Jaeger collector |
| Jaeger | `jaeger` | Distributed tracing, Badger storage |
| Headlamp | `headlamp` | K8s dashboard, OIDC via kgateway |

## Messaging

| Component | Namespace | Notes |
|-----------|-----------|-------|
| Strimzi (Kafka) | `kafka` | KRaft mode, 3 combined nodes |
| NATS | `nats` | 3-node, JetStream enabled |
| RabbitMQ | `rabbitmq` | OLM-managed, 3 replicas |

## Serverless

| Component | Namespace | Notes |
|-----------|-----------|-------|
| Knative Operator | `knative-operator` | Manages Serving + Eventing |
| Knative Serving | `knative-serving` | `*.kn.wcloud.sh`, net-gateway-api |
| Knative Eventing | `knative-eventing` | Kafka as default broker |

## Dev platform

| Component | Namespace | Notes |
|-----------|-----------|-------|
| Coder | `coder` | CDEs, Percona PostgreSQL backend |
| Temporal | `temporal-system` | Durable workflows, OIDC, PostgreSQL |
| GitHub ARC | `arc-systems` | AMD64 + ARM64 scale sets (0–4 each) |
| HCP Terraform agents | `tfc-operator-system` | AMD64 + ARM64 pools (3–10 each) |
| Camel K + Kaoto | `camel` | Cloud-native integration, OLM |

## Argo

| Component | Namespace | Notes |
|-----------|-----------|-------|
| ArgoCD | `argo` | GitOps, Keycloak OIDC |
| Argo Workflows | `argo` | DAG workflows, Keycloak OIDC |
| Argo Events | `argo` | Event-driven triggers |

## Databases & caching

| Component | Namespace | Notes |
|-----------|-----------|-------|
| Percona PostgreSQL Operator | `postgres` | Managed PostgreSQL clusters |
| Dragonfly Operator | `dragonfly` | Redis-compatible, in-memory |

## Autoscaling

| Component | Namespace | Notes |
|-----------|-----------|-------|
| VPA | — | `InPlaceOrRecreate` update mode |
| KEDA | `keda` | Event-driven autoscaling |

## Other

| Component | Namespace | Notes |
|-----------|-----------|-------|
| Kubernetes Replicator | `kube-system` | Secret/ConfigMap replication |
| OLM | `olm` | Operator catalog |
| Metrics Server | `kube-system` | Core resource metrics |
| HCP Terraform Operator | `tfc-operator-system` | Terraform workspace management |
