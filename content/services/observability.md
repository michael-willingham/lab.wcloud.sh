---
title: "Observability"
date: 2026-03-01
weight: 20
tags: ["observability", "grafana", "victoriametrics", "jaeger", "opentelemetry"]
summary: "Metrics, tracing, and dashboards — VictoriaMetrics, Grafana, Jaeger, Hubble, and Headlamp."
---

## Metrics: VictoriaMetrics

The [VictoriaMetrics Kubernetes Stack (VMKS)](https://docs.victoriametrics.com/guides/getting-started-with-vm-operator/) replaces the traditional Prometheus stack. It includes:

- **VictoriaMetrics** as the metrics backend (`metrics.wcloud.sh`)
- **Grafana** for dashboards (`grafana.wcloud.sh`), authenticated via Keycloak OIDC
- **AlertManager** for alert routing (`alerts.wcloud.sh`)

## Tracing: Jaeger + OpenTelemetry

The [OpenTelemetry Operator](https://opentelemetry.io/docs/kubernetes/operator/) manages a [Jaeger](https://www.jaegertracing.io/) collector deployed as an `OpenTelemetryCollector` CR. Uses [Badger](https://github.com/dgraph-io/badger) embedded storage on a 10Gi PVC with 7-day trace retention.

**URL:** `jaeger.wcloud.sh`

## Network observability: Hubble

[Hubble](https://docs.cilium.io/en/stable/observability/hubble/) provides L3/L4/L7 network flow visibility, powered by Cilium's eBPF datapath. Useful for debugging network policies and understanding traffic patterns.

**URL:** `hubble.wcloud.sh`

## Kubernetes dashboard: Headlamp

[Headlamp](https://headlamp.dev/) serves as the Kubernetes dashboard with plugins for:
- Flux
- cert-manager
- KEDA

Authentication is handled via a kgateway OAuth2 policy that redirects to Keycloak.

**URL:** `k8s.wcloud.sh`
