---
title: "Dev Platform"
date: 2026-03-01
weight: 50
tags: ["coder", "temporal", "arc", "terraform", "camel-k"]
summary: "Coder CDEs, Temporal workflows, GitHub Actions runners, Terraform agents, and Camel K."
---

## Coder

[Coder](https://coder.com/) provides Cloud Development Environments running on the cluster. Workspaces spin up as Kubernetes pods with persistent storage.

- **Backend:** Percona PostgreSQL
- **URL:** `coder.wcloud.sh`

## Temporal

[Temporal](https://temporal.io/) provides durable workflow execution — workflows that survive process crashes, node failures, and deployments.

- **3 server replicas** + **2 web UI replicas**
- **Backend:** Percona PostgreSQL
- **Auth:** Keycloak OIDC
- **URL:** `temporal.wcloud.sh`
- **VPA-managed** resource allocation

## GitHub Actions runners (ARC)

[Actions Runner Controller](https://github.com/actions/actions-runner-controller) provides self-hosted GitHub Actions runners:

| Scale set | Architecture | Replicas |
|-----------|-------------|----------|
| AMD64 | x86_64 | 0–4 (autoscaled) |
| ARM64 | aarch64 | 0–4 (autoscaled) |

Separate scale sets for each architecture reflect the mixed AMD64/ARM64 nature of the cluster. Authentication uses a GitHub App.

## HCP Terraform agents

Self-hosted [Terraform Cloud agents](https://developer.hashicorp.com/terraform/cloud-docs/agents) for running Terraform plans and applies:

| Pool | Architecture | Replicas |
|------|-------------|----------|
| AMD64 | x86_64 | 3–10 |
| ARM64 | aarch64 | 3–10 |

Managed by the [HCP Terraform Operator](https://developer.hashicorp.com/terraform/cloud-docs/integrations/kubernetes/setup).

## Camel K + Kaoto

[Apache Camel K](https://camel.apache.org/camel-k/) provides cloud-native integration — deploy integration routes as lightweight serverless-style containers.

[Kaoto](https://kaoto.io/) adds a visual low-code designer for building integration flows.

Both deployed via OLM. **URL:** `camel.wcloud.sh`
