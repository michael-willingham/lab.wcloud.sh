---
title: "Argo Suite"
date: 2026-03-01
weight: 60
tags: ["argocd", "argo-workflows", "argo-events"]
summary: "ArgoCD for application GitOps, Argo Workflows for DAG execution, Argo Events for triggers."
---

All three Argo components run in the `argo` namespace.

## ArgoCD

[ArgoCD](https://argo-cd.readthedocs.io/) handles application-level GitOps deployments. While FluxCD manages the infrastructure layer (operators, CRDs, networking), ArgoCD is available for application workloads.

- **Auth:** Keycloak OIDC
- **URL:** `argo-cd.wcloud.sh`

## Argo Workflows

[Argo Workflows](https://argo-workflows.readthedocs.io/) provides DAG-based workflow execution on Kubernetes. Workflows are defined as Kubernetes CRs and execute as pods.

- **Auth:** Keycloak OIDC
- **URL:** `argo-workflows.wcloud.sh`

## Argo Events

[Argo Events](https://argoproj.github.io/argo-events/) connects event sources to workflow triggers. Events from webhooks, message queues, or schedules can trigger Argo Workflows or Kubernetes resources.
