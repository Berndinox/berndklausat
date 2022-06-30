---
slug: kubernetes-on-the-edge
draft: false
title: Kubernetes on the edge
date: 2022-06-24T12:40:32.169Z
tags:
  - Edge
  - Kubernetes
  - Distributed-System
  - Master
---

## Edge-Computing with Kubernetes?

With the trend of 5G, the distributed processing of large amounts of data at the network edge is becoming an important part of modern IT environments. Kubernetes provides a ready-made construction kit to dynamically launch and scale workloads. In this respect, it is also excellently suited for use in edge-computing. However, there are different methods to build such a Kubernetes cluster suitable for such purposes.

- Default Cluster: with the ability to compensate latencies accordingly
- Distributed Clusters: which are controlled by a central instance

**The question is, which variant is better, or even better: which variant is the right one for my application?**

This question is answered in the following thesis, which deals with the topic in detail. Specifically, the tools KubeEdge (Default) and KubeFed (Distribute) are compared.

Thesis Link: **[Berndinox/K8sEdge](https://github.com/Berndinox/K8sEdge/blob/main/MCS_K8sEdge_Bernd_KLAUS.pdf)**

In order to find the right environment for his individual application, a decision catalog was created, which can be used for the evaluation. Also, the weighting of each category can be adjusted to its own needs.


