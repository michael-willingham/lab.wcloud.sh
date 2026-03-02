---
title: "Multi-Node GPU Training with NCCL"
date: 2026-02-28
draft: true
tags: ["nccl", "gpu", "rdma", "distributed-training", "dgx-spark"]
categories: ["learning"]
series: ["RDMA on Kubernetes"]
summary: "Setting up NCCL for distributed training across two DGX Sparks — GPU-Direct RDMA, environment variables, and the quest for benchmarks."
---

> **Note:**
> **This post is a work in progress.** I'm still working through the NCCL setup and benchmarking. I'll update it as I make progress.

## What is NCCL?

[NCCL (NVIDIA Collective Communications Library)](https://developer.nvidia.com/nccl) is NVIDIA's library for multi-GPU and multi-node collective operations — all-reduce, all-gather, broadcast, etc. It's the communication backbone for distributed training frameworks like PyTorch DDP and DeepSpeed.

NCCL automatically detects available communication channels (NVLink, PCIe, Ethernet, InfiniBand/RoCE) and chooses the optimal topology. The goal is to get it to detect and use the 200 Gbps RDMA link between the two DGX Sparks.

## The setup

### Pod configuration

Each training pod needs:
- A GPU (`nvidia.com/gpu: 1`)
- An SR-IOV VF on the QSFP link (`nvidia.com/cx7_qsfp: 1`)
- A Multus annotation to attach the QSFP network

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    k8s.v1.cni.cncf.io/networks: dgx-qsfp
spec:
  containers:
  - name: training
    image: nvcr.io/nvidia/pytorch:24.07-py3
    resources:
      requests:
        nvidia.com/gpu: 1
        nvidia.com/cx7_qsfp: 1
      limits:
        nvidia.com/gpu: 1
        nvidia.com/cx7_qsfp: 1
    env:
    - name: NCCL_SOCKET_IFNAME
      value: "net1"
    - name: NCCL_IB_DISABLE
      value: "0"
    - name: NCCL_NET_GDR_LEVEL
      value: "5"
    - name: NCCL_DEBUG
      value: "INFO"
```

### NCCL environment variables

| Variable | Value | Purpose |
|----------|-------|---------|
| `NCCL_SOCKET_IFNAME` | `net1` | Use the Multus RDMA interface, not `eth0` |
| `NCCL_IB_DISABLE` | `0` | Enable InfiniBand/RoCE transport |
| `NCCL_NET_GDR_LEVEL` | `5` | Enable GPU-Direct RDMA |
| `NCCL_DEBUG` | `INFO` | Log transport selection and topology |

### What GPU-Direct RDMA level 5 means

`NCCL_NET_GDR_LEVEL=5` tells NCCL to use GPU-Direct RDMA at the highest level — the GPU reads/writes directly to/from the NIC's registered memory, bypassing CPU and system memory entirely.

On a discrete GPU system, this requires specific PCIe topology (the GPU and NIC need to be on the same PCIe root complex or closer). On the DGX Spark with unified memory, the topology question is different — I'm still investigating how GPU-Direct behaves when GPU and CPU share the same physical memory.

## Benchmarking plan

The standard NCCL benchmark suite is [nccl-tests](https://github.com/NVIDIA/nccl-tests). The key test is `all_reduce_perf`, which measures all-reduce bandwidth across different message sizes.

What I want to measure:
1. **Peak bandwidth** — how close to 200 Gbps (25 GB/s) can all-reduce achieve?
2. **Message size curve** — at what message size does bandwidth saturate?
3. **GPU-Direct vs non-GPU-Direct** — what's the performance difference?
4. **Latency** — small-message latency for gradient synchronization

## Current status

I have the pods running with the correct resources and interfaces attached. NCCL detects the `net1` interface. I'm currently working through:

- Verifying that RDMA operations work correctly between the pods (ibv_rc_pingpong)
- Getting NCCL to select the RoCE transport (vs falling back to TCP sockets)
- Running the first all_reduce_perf benchmarks

Results coming soon.
