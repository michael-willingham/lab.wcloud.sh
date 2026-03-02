---
title: "Serverless: Knative"
date: 2026-03-01
weight: 40
tags: ["knative", "serverless", "kafka"]
summary: "Knative Serving for scale-to-zero workloads, Eventing backed by Kafka."
---

## Knative Serving

[Knative Serving](https://knative.dev/docs/serving/) provides serverless compute — containers that auto-scale based on traffic, including scaling to zero when idle.

- **Domain template:** `{name}-{namespace}.kn.wcloud.sh`
- **Networking:** `net-gateway-api` bridges Knative to kgateway, so Knative Services get real Gateway API HTTPRoutes
- **Managed by:** [Knative Operator](https://knative.dev/docs/install/operator/knative-with-operators/)

## Knative Eventing

[Knative Eventing](https://knative.dev/docs/eventing/) provides event-driven architecture with a [CloudEvents](https://cloudevents.io/) contract.

The default broker is **Kafka-backed** via `eventing-kafka-broker`, which means:

- Events are durably stored in Kafka topics
- Triggers subscribe to event types and route them to Knative Services
- The full Strimzi Kafka cluster provides the backbone

{{< mermaid >}}
graph LR
    Source["Event Source"] -->|CloudEvent| Broker["Knative Broker<br/>(Kafka-backed)"]
    Broker -->|Trigger filter| Svc1["Knative Service A"]
    Broker -->|Trigger filter| Svc2["Knative Service B"]
{{< /mermaid >}}

This integration is one of the more elegant parts of the cluster — serverless functions triggered by durable event streams.
