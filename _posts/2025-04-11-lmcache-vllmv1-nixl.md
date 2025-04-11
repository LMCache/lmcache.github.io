---
layout: post
title: "Shaping NIXL-based PD Disaggregation in vLLM V1"
thumbnail-img: https://github.com/user-attachments/assets/d969ca83-cb3b-4a90-8859-880264d809ed
share-img: https://github.com/user-attachments/assets/d969ca83-cb3b-4a90-8859-880264d809ed
author: LMCache Team
image: https://github.com/user-attachments/assets/d969ca83-cb3b-4a90-8859-880264d809ed
---
<br>

### Highlights:
Today, LMCache shares two key designs in LLM infrastructure for disaggregated prefill and more:
- **LMCache x vLLM V1**: The **first** PD disaggregation for **vLLM V1**. [**Link to PR**](https://github.com/vllm-project/vllm/pull/15960/)
- **LMCache x NIXL**: **Support for NVIDIA's NIXL** enabling ultra-fast KV cache transfers across GPUs and nodes. [**Link to PR**](https://github.com/LMCache/LMCache/pull/446)

Together, these updates mark a pivotal leap forward in *PD disaggregation* for vLLM, towards better system flexibility and multi-node scale-out capabilities.

A high-level architecture diagram of "vLLM V1 + NIXL + LMCache" integration:

<img width="414" alt="image" src="https://github.com/user-attachments/assets/d969ca83-cb3b-4a90-8859-880264d809ed" />




## vLLM V1 Gets a Major Upgrade with LMCache and NIXL

vLLM V1 has been a massive step forward for the LLM ecosystem. 
With its cleaner codebase, modular architecture, and performance optimizations, it quickly became the go-to inference engine for high-throughput and low-latency LLM serving. 
However, one critical feature was notably absent --- support for a highly performant **KV cache layer**. 
In the earlier vLLM V0, the KV connector was central to optimizations such as KV reuse, memory-efficient context handling, and dynamic batch execution. 
Yet, this feature was left behind in the V1 refactor. That changes today.

With this release, LMCache Lab introduces **the first official KV cache layer integration for vLLM V1**. 
Like the KV connector in vLLM V0, this integration exposes a series of interfaces that allow external systems and developers to **extract/inject KV cache** entries directly from/to vLLM’s internal **paged KV memory**. 
This paves the way for a new KV cache layer, and it's critical for workloads that depend on multi-user state sharing and fast user context switching across sessions and nodes.

## LMCache’s Two Core KV Cache Functions: Storage and Transport

LMCache now supports two major modes of operation (as depicted in the following diagram):

<img width="732" alt="image" src="https://github.com/user-attachments/assets/5c8d1086-b4c7-44b9-bab7-bcd4c5776b69" />



### 1. **Storage: DB-Style Loading and Offloading**

This mode focuses on **persistent KV cache storage**. 
It’s perfect for workloads like **KV cache sharing** across user sessions or even between nodes in a cluster. 
LMCache maintains a database-style abstraction that can offload hot KV entries from GPU memory to CPU memory, or even to disk, using custom eviction and prefetch strategies. 
This enables long-term cache persistence and retrieval, dramatically increasing reuse rates for popular user prompts or conversation histories.

The new LMCache integration is designed to align with vLLM V1’s advanced memory features. It works seamlessly with **prefix caching**, **chunked prefill**, and the **paged memory manager**. 

<!-- But it doesn’t stop there. LMCache introduces a new set of APIs that extend the KV cache abstraction far beyond what was possible in v0. These include:-->

<!-- - **Layer-by-layer KV injection**: Only need attention layers? You can inject them selectively. -->
<!-- - **Asynchronous KV extraction**: Extract KV entries without blocking the main inference thread. -->
<!-- - **KV cache prefetching**: Predict and load required KV chunks before the model even needs them. -->

Moreover, LMCache’s connector is now officially integrated into upstream vLLM, providing a plug-and-play experience for developers and researchers alike. No hacks. No forks. Just clean and efficient extensions to the vLLM core.

### 2. **Transport: P2P-Style Direct Transfer**

The second mode targets **disaggregated prefill** scenarios, where KV cache needs to be fetched from remote nodes in real-time. Instead of recomputing context tokens locally, LMCache enables **peer-to-peer KV transfers**, cutting down on redundant compute and unlocking cross-node cache reuse. This feature is crucial for multi-GPU, multi-node serving environments that demand extreme efficiency.

To push KV cache transport even further, LMCache now includes **first-class support for NIXL**, NVIDIA’s new communication abstraction designed for heterogeneous hardware. NIXL supports **NVLink**, **RDMA-capable NICs**, and **GPU Direct Storage**, offering unmatched bandwidth and latency across devices.

LMCache’s modular transport layer plugs directly into NIXL’s communication stack, allowing it to dynamically select the fastest available data path between GPUs, CPUs, and storage. Whether it’s a local multi-GPU system or a datacenter-scale cluster, LMCache with NIXL delivers blazing fast KV cache movement, often faster than recomputing even a single token’s attention.

This unlocks new LLM serving paradigms: real-time distributed KV cache retrieval, zero-compute context reconstruction, and even **memory disaggregation** at the hardware level.

---

## Stay Tuned for More Performance Benchmarking

In this blog, we only discussed the design of the newest integration. We will share the performance optimizations implemented along with this effort and release benchmarking numbers in the coming blogs. Follow us on LinkedIn and Twitter(X) to learn more next week!

## Links
- **LMCache Github: [https://github.com/LMCache/LMCache](https://github.com/LMCache/LMCache)**
- **Chat with the Developers** **[Interest Form](https://forms.gle/mQfQDUXbKfp2St1z7)**
- **LMCache [slack](https://join.slack.com/t/lmcacheworkspace/shared_invite/zt-2viziwhue-5Amprc9k5hcIdXT7XevTaQ)**
- **vLLM Production-Stack [channel](https://vllm-dev.slack.com/archives/C089SMEAKRA)**
