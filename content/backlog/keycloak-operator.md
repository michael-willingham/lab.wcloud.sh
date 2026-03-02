---
title: "Custom Keycloak Client Operator"
date: 2026-03-01
tags: ["keycloak", "kubernetes", "golang", "operator"]
summary: "Build a Kubernetes operator in Go to declaratively manage Keycloak OIDC clients via CRDs."
---

## The problem

Every time I add a new service that needs OIDC, I have to manually create a Keycloak client — essentially running a script against the Keycloak admin API. The client ID, redirect URIs, and scopes aren't tracked in Git. It works, but it's hacky and fragile.

I already have [7 OIDC consumers](/services/keycloak/) in the cluster, and the list keeps growing. Each one requires creating the client, setting redirect URIs, configuring scopes, and syncing the client secret into Infisical so ESO can distribute it. None of that is declarative or gitops-friendly.

## What I want to build

A Kubernetes operator in Go that watches a CRD like:

```yaml
apiVersion: keycloak.wcloud.sh/v1alpha1
kind: KeycloakClient
metadata:
  name: grafana
  namespace: keycloak
spec:
  realmRef: wcloud
  clientId: grafana
  protocol: openid-connect
  redirectUris:
    - "https://grafana.wcloud.sh/login/generic_oauth"
  defaultScopes:
    - openid
    - profile
    - email
```

The operator would reconcile this against the Keycloak admin API — creating, updating, or deleting clients to match the desired state.

## Why I want to do this

Two reasons:

1. **Solve a real problem** — get Keycloak client management into Git where it belongs, alongside everything else in the cluster.

2. **Learn to build a proper Kubernetes operator** — I've never written one from scratch. I want to learn the modern Go stack: kubebuilder or operator-sdk, controller-runtime, custom resource definitions, finalizers, status conditions, the whole reconciliation loop. This feels like the right project for it because the scope is well-defined and I'll actually use it every day.

## Open questions

- **kubebuilder vs operator-sdk** — which one is the better starting point in 2026? They share a lot of code (both use controller-runtime), but I need to pick one to scaffold with.
- **Client secret lifecycle** — should the operator generate the client secret and write it to a Kubernetes Secret, or should it integrate with Infisical/ESO? Probably both as options.
- **Existing operators** — there are community Keycloak operators that handle client management (like the one from Epam). Worth evaluating whether one of those is good enough, or if the learning value justifies building my own.
- **Realm management** — should I keep scope tight (just clients) or also handle realm configuration, identity providers, and client scopes?
