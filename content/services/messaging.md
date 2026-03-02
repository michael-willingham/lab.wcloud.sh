---
title: "Messaging"
date: 2026-03-01
weight: 30
tags: ["kafka", "nats", "rabbitmq", "messaging"]
summary: "Three message brokers for three different patterns — Kafka, NATS, and RabbitMQ."
---

## Why three brokers?

Each broker excels at a different messaging pattern:

| Broker | Strength | Use case |
|--------|----------|----------|
| **Kafka** | Durable event streaming, replay, ordering | Event sourcing, Knative Eventing backend |
| **NATS** | Ultra-low latency, lightweight pub/sub | Service-to-service messaging, request/reply |
| **RabbitMQ** | Flexible routing, dead-letter queues | Task queues, complex routing topologies |

Running all three is partly practical, partly educational — understanding the trade-offs between them is the point.

## Kafka (Strimzi)

[Strimzi](https://strimzi.io/) manages a Kafka cluster:

- **KRaft mode** — no ZooKeeper dependency
- **3 combined controller+broker nodes** — each node runs both roles
- **Replication factor 3, min ISR 2** — tolerates one node failure
- **50 GiB Longhorn storage** per node
- **Kafbat UI** at `kafka.wcloud.sh` with Keycloak OIDC auth

Kafka also serves as the **default broker backend for Knative Eventing** — CloudEvents flow through Kafka topics.

## NATS

[NATS](https://nats.io/) runs as a 3-node cluster with JetStream enabled:

- **JetStream** provides persistence, exactly-once delivery, and key-value store
- **20 GiB Longhorn storage** per node
- Lightweight and fast — ideal for internal service communication

## RabbitMQ

[RabbitMQ](https://www.rabbitmq.com/) is deployed via the RabbitMQ Cluster Operator (OLM-managed):

- **3 replicas** with topology spread constraints across nodes
- **20 GiB Longhorn storage** per node
- **OAuth2 backend plugin** — authenticates management UI users via Keycloak
- **Management UI** at `rabbitmq.wcloud.sh`
