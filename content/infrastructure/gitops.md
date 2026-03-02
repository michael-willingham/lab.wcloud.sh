---
title: "GitOps"
date: 2026-03-01
weight: 50
tags: ["gitops", "fluxcd"]
summary: "FluxCD-driven reconciliation — if it's not in Git, it doesn't exist."
---

## FluxCD

[FluxCD](https://fluxcd.io/) is the GitOps engine for the cluster. It continuously reconciles the desired state in Git with the actual state in the cluster.

### Components

- helm-controller
- kustomize-controller
- notification-controller
- source-controller

### Workflow

The workflow is strict:

1. Edit YAML in the Git repository
2. Commit and push to `master`
3. FluxCD detects the change and reconciles
4. The cluster converges to the desired state

`kubectl apply` is never used directly. If something needs to change, it goes through Git.

### Dependency management

FluxCD Kustomizations are organized with explicit `dependsOn` relationships. See [Architecture](/infrastructure/architecture/) for the full dependency graph.

### Automated updates

[Renovate](https://docs.renovatebot.com/) runs on the repository to automatically propose updates to Helm chart versions and container image tags. Updates arrive as pull requests for review before merging.

### Monitoring

```bash
# Check Kustomization status
flux get kustomizations

# Check HelmRelease status across all namespaces
flux get helmreleases -A

# Follow reconciliation logs
flux logs --all-namespaces --follow
```
