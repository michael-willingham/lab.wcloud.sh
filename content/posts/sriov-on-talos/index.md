---
title: "SR-IOV on Talos: Why the Operator Doesn't Work"
date: 2026-02-15
tags: ["sr-iov", "talos", "networking", "dgx-spark"]
categories: ["learning"]
series: ["RDMA on Kubernetes"]
summary: "The SR-IOV Network Operator crashes on Talos Linux. Here's the standalone device plugin I built instead."
---

## The problem

The standard way to expose SR-IOV devices in Kubernetes is the [SR-IOV Network Operator](https://github.com/k8snetworkplumbingwg/sriov-network-operator). It handles:

1. Discovering SR-IOV-capable NICs
2. Creating Virtual Functions (VFs) on those NICs
3. Running the SR-IOV device plugin to advertise VFs to Kubernetes
4. Managing NetworkAttachmentDefinitions for Multus

Sounds perfect. Except it **crashes on Talos Linux**.

### The root cause

The operator's `sriov-config-daemon` assumes it can write to `/etc/sriov-operator/` and other paths on the host filesystem. On a normal Linux distro, this works fine. On Talos, the root filesystem is **read-only and immutable**. There's no `/etc/sriov-operator/`, and you can't create it.

The daemon enters a crash loop:

```
error creating /etc/sriov-operator/config.yaml: read-only file system
```

I looked into patching the operator to use a different path, but the assumption of a writable `/etc/` is baked deep into the codebase. It wasn't a quick fix.

## The solution: standalone device plugin

Instead of fighting the operator, I stripped it down to just the parts I needed and built a standalone DaemonSet.

### Step 1: Create VFs with an init container

SR-IOV Virtual Functions are created by writing to sysfs, which **is** writable on Talos (it's a kernel interface, not a filesystem):

```bash
# Enable SR-IOV and create 4 VFs on the ConnectX-7
echo 4 > /sys/class/net/<pf-name>/device/sriov_numvfs
```

The init container:
1. Finds the ConnectX-7 PF (Physical Function) on the DGX Spark nodes
2. Writes to `sriov_numvfs` to create 4 Virtual Functions
3. Waits for the VF netdevs to appear

This runs as a privileged init container in the DaemonSet, with a `nodeSelector` targeting only the DGX Spark nodes.

### Step 2: Run the device plugin

The main container in the DaemonSet runs the [SR-IOV Network Device Plugin](https://github.com/k8snetworkplumbingwg/sriov-network-device-plugin) directly, configured to discover the VFs and advertise them as `nvidia.com/cx7_qsfp` resources.

The device plugin config:

```json
{
  "resourceList": [
    {
      "resourceName": "cx7_qsfp",
      "resourcePrefix": "nvidia.com",
      "selectors": {
        "vendors": ["15b3"],
        "devices": ["101e"],
        "drivers": ["mlx5_core"],
        "pfNames": ["<pf-name>#0-3"]
      }
    }
  ]
}
```

This discovers VFs 0–3 on the ConnectX-7 PF and registers them with Kubernetes.

### Step 3: NetworkAttachmentDefinition

A `NetworkAttachmentDefinition` tells Multus how to configure the secondary interface:

```yaml
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: dgx-qsfp
  namespace: kube-system
spec:
  config: |
    {
      "cniVersion": "0.3.1",
      "name": "dgx-qsfp",
      "type": "sriov",
      "ipam": {
        "type": "host-local",
        "subnet": "10.100.0.0/24"
      },
      "rdma": true
    }
```

RDMA is explicitly enabled, and IPAM assigns addresses from `10.100.0.0/24`.

## The result

After deploying the DaemonSet:

```
$ kubectl get nodes -o custom-columns=NAME:.metadata.name,QSFP:.status.allocatable.nvidia\\.com/cx7_qsfp
NAME              QSFP
talos-76w-3r0     <none>
talos-7aj-lwl     4
talos-ysi-4k0     4
```

The DGX Spark nodes each advertise 4 `nvidia.com/cx7_qsfp` resources. Pods can request one to get a secondary network interface on the 200 Gbps QSFP link.

## Lessons learned

1. **Don't fight the operator.** If an operator assumes a writable rootfs and your OS doesn't have one, building a simpler standalone solution is faster than patching the operator.
2. **sysfs is your friend on Talos.** The kernel interface is always writable, even when the root filesystem isn't. SR-IOV VF creation works through sysfs, so we can work around the read-only `/etc/` restriction.
3. **The device plugin is the easy part.** The SR-IOV device plugin is a relatively simple binary that watches sysfs for VFs and registers them with the kubelet. The complexity in the operator is mostly around automation that we handled with a simpler init container approach.
4. **Init containers for hardware setup.** Using an init container to configure SR-IOV VFs is clean — it runs once at pod startup, before the device plugin starts discovering VFs. The ordering guarantee is exactly what we need.
