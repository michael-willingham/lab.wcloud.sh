---
title: "About"
description: "About this site and the cluster behind it."
---

## About this site

This is a documentation site and learning journal for **willingham-k8s**, a homelab Kubernetes cluster I built to learn about GPU infrastructure, RDMA networking, and platform engineering at a level deeper than what you can get from managed cloud services.

The cluster runs real workloads — development environments, CI runners, workflow orchestration, event-driven processing — on hardware I own and manage. When something breaks, there's no support ticket to file. That's the point.

## About the cluster

- **3 nodes:** 1 custom AMD64 PC + 2 NVIDIA DGX Sparks (ARM64)
- **448 GB RAM, ~3.4 PFLOPS FP4** AI compute
- **200 Gbps RDMA interconnect** between the DGX Sparks
- **Talos Linux** — immutable, API-driven, no SSH
- **GitOps via FluxCD** — every change goes through Git
- **Keycloak SSO** for every service

See the [infrastructure](/infrastructure/) section for the full breakdown.

## About me

I'm Michael Willingham — software engineer on the Fleet Automation team at [NorthMark Compute & Cloud (NMC2)](https://nmc2.com), a high-performance compute infrastructure company building one of the largest private GPU clusters in the world at their 400MW facility in Spartanburg, SC. Outside of work, I run this homelab to go deeper on the same kinds of problems: Kubernetes, GPU scheduling, RDMA networking, and platform engineering.

- [GitHub](https://github.com/michael-willingham)
- [LinkedIn](https://www.linkedin.com/in/michaelwillingham/)

## About the tech

This site is built with [Hugo](https://gohugo.io/) and the [Blowfish](https://blowfish.page/) theme, deployed to [GitHub Pages](https://pages.github.com/) via GitHub Actions. Source is at [willingham-cloud/lab.wcloud.sh](https://github.com/willingham-cloud/lab.wcloud.sh).
