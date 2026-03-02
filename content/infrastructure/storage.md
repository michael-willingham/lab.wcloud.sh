---
title: "Storage"
date: 2026-03-01
weight: 40
tags: ["storage", "longhorn"]
summary: "Longhorn v2 data engine for distributed block storage."
---

## Longhorn

[Longhorn](https://longhorn.io/) provides distributed block storage with replicated volumes and snapshots.

### v2 data engine

The cluster runs with the **v2 data engine enabled** (`defaultSettings.v2DataEngine: true`). The v2 engine uses SPDK (Storage Performance Development Kit) for improved I/O performance compared to the legacy iSCSI-based v1 engine.

### Resource tuning

On the DGX Spark nodes (20 cores each), the default Longhorn instance manager CPU reservation was too aggressive. The setting `guaranteedInstanceManagerCPU` is tuned to `{"v1":"5","v2":"5"}` to reduce per-instance-manager reservation.

### Consumers

Stateful workloads using Longhorn PVCs:

| Workload | Storage per replica |
|----------|-------------------|
| Kafka (Strimzi, 3 brokers) | 50 GiB |
| NATS (3 nodes) | 20 GiB |
| RabbitMQ (3 replicas) | 20 GiB |
| Grafana | varies |

### Web UI

The Longhorn dashboard is available at `longhorn.wcloud.sh` for volume management, snapshot creation, and monitoring.
