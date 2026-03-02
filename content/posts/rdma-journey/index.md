---
title: "My RDMA Journey with DGX Spark"
date: 2026-03-01
tags: ["rdma", "dgx-spark", "nvidia", "networking"]
categories: ["learning"]
series: ["RDMA on Kubernetes"]
summary: "Why I bought two DGX Sparks, what RDMA actually is, and where I'm at with getting it working in Kubernetes."
---

## Why DGX Spark?

I'd been running a single-node homelab for a while — the Ryzen 9 build. It handled everything I threw at it, but I kept bumping into the same wall: **multi-node GPU training is fundamentally different from single-node**, and I couldn't learn it without actually doing it.

The DGX Spark hit a sweet spot:
- Desktop form factor (no rack, no 240V circuit)
- **Grace Blackwell SoC** — ARM64 CPU and Blackwell GPU with 128 GB unified memory
- **ConnectX-7 NIC** — 200 Gbps, SR-IOV, RDMA-capable
- The GPU and NIC can talk directly via GPU-Direct RDMA

I bought two. Connected them with a QSFP56 DAC cable. And then the real learning started.

## What is RDMA?

RDMA (Remote Direct Memory Access) lets one machine read from or write to another machine's memory **without involving the remote CPU**. The NIC handles the transfer directly.

In traditional networking:
1. Application calls `send()`
2. Data copies from user space to kernel buffer
3. Kernel hands it to the NIC
4. NIC transmits
5. Remote NIC receives
6. Remote kernel copies to kernel buffer
7. Remote kernel copies to user-space application

With RDMA:
1. Application registers a memory region with the NIC
2. NIC reads directly from application memory and transmits
3. Remote NIC writes directly to the remote application's memory

No kernel involvement on the data path. No copies. The CPU is free to do other work while data moves at line rate.

### RoCE v2

There are several RDMA transports. The ConnectX-7 supports **RoCE v2 (RDMA over Converged Ethernet)**, which runs RDMA over standard Ethernet with UDP encapsulation. This means I don't need InfiniBand hardware — just Ethernet (or in my case, a direct QSFP cable).

## Why this matters for GPU training

Distributed training involves two main communication patterns:

1. **All-Reduce** — every GPU needs to aggregate gradients from every other GPU after each training step
2. **All-Gather** — redistributing updated parameters to all GPUs

With GPU-Direct RDMA, NCCL (NVIDIA's collective communication library) can move data between GPUs on different nodes **without touching the CPU or system memory**:

```
GPU (Node 1) → NIC (Node 1) → [200 Gbps link] → NIC (Node 2) → GPU (Node 2)
```

The GPU's memory is directly accessible to the NIC. No staging through CPU memory. This is critical for training performance — the faster the inter-node communication, the less time GPUs spend idle waiting for gradient synchronization.

## Where I'm at

### Working

- Both DGX Sparks are running Talos Linux and joined to the Kubernetes cluster
- The ConnectX-7 NICs are detected and the QSFP link is up at 200 Gbps
- SR-IOV Virtual Functions are created via a custom init container (Talos's read-only rootfs prevents the standard SR-IOV Network Operator from working)
- Multus attaches a secondary `net1` interface to pods, connected to the QSFP link
- Basic pod-to-pod communication works across the 200 Gbps link

### In progress

- Validating RDMA operations (ibv_rc_pingpong, etc.) between pods
- Getting NCCL to detect and use the RDMA interface
- Running NCCL benchmarks (all_reduce_perf) to measure actual throughput

### Open questions

- What's the real-world NCCL throughput I should expect on this link? The theoretical max is 200 Gbps (25 GB/s), but protocol overhead, message size, and GPU-Direct efficiency all affect this.
- Do I need to tune any RoCE v2 parameters (ECN, PFC) for optimal performance on a direct point-to-point link? Most RoCE tuning guides assume a switched fabric.
- How does the unified memory architecture of the GB10 interact with GPU-Direct RDMA? The GPU and CPU share the same physical memory — does this change how GPU-Direct works compared to discrete GPU systems?

These are the questions I'm actively investigating. I'll update this series as I figure them out.
