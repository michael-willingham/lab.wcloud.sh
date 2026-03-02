---
title: "Identity: Keycloak"
date: 2026-03-01
weight: 10
tags: ["keycloak", "oidc", "sso"]
summary: "Centralized OIDC SSO for every service in the cluster."
---

## Overview

[Keycloak](https://www.keycloak.org/) provides centralized identity management via the `wcloud` realm. Nearly every web UI in the cluster authenticates through Keycloak using OIDC.

Deployed via the Keycloak Operator (OLM-managed) in the `keycloak` namespace.

**URL:** `keycloak.wcloud.sh`

## OIDC consumers

| Service | Auth method |
|---------|------------|
| ArgoCD | OIDC client |
| Argo Workflows | OIDC client |
| Grafana | OIDC client |
| Temporal | OIDC client |
| Kafbat (Kafka UI) | OIDC client |
| Headlamp | kgateway OAuth2 policy → Keycloak |
| RabbitMQ | OAuth2 backend plugin |

## External identity provider

Google OAuth is configured as an upstream identity provider in the `wcloud` realm. Users can sign in with their Google account, and Keycloak handles the federation and token issuance.

## Secrets

All OIDC client secrets are managed via the External Secrets Operator, synced from Infisical. No secrets live in Git.
