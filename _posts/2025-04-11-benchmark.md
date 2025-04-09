---
layout: post
title: "The Next-Gen KV Cache Layer: LMCache x vLLM V1 x NIXL"
thumbnail-img: 
share-img: 
author: LMCache Team
image: 
---
<be>

### Executive Summary:
Today, LMCache announces two groundbreaking advancements in LLM system infrastructure:
- **LMCache x vLLM V1**: The **first** KV cache layer integration with **vLLM V1**.
- **LMCache x NIXL**: Full **support for NVIDIA's NIXL** enabling ultra-fast KV cache transfers across GPUs and nodes.

Together, these updates mark a pivotal leap forward in inference performance, system flexibility, and multi-node scale-out capabilities.

A high-level architecture diagram of "vLLM V1 + LMCache + NIXL" integration:

<img width="414" alt="image" src="https://github.com/user-attachments/assets/13aa2c9c-517a-44ab-98e5-032e379ba877" />



## vLLM V1 Gets a Major Upgrade with LMCache and NIXL

vLLM V1 has been a massive step forward for the LLM ecosystem. 
With its cleaner codebase, modular architecture, and performance optimizations, it quickly became the go-to inference engine for high-throughput and low-latency LLM serving. 
However, one critical feature was notably absent --- support for a highly performant **KV cache layer**. 
In the earlier vLLM V0, the KV connector was central to optimizations such as KV reuse, memory-efficient context handling, and dynamic batch execution. 
Yet, this feature was left behind in the V1 refactor. That changes today.

With this release, LMCache Lab introduces **the first official KV cache layer integration for vLLM V1**. 
Like the KV connector in vLLM V0, this integration exposes a series of interfaces that allow external systems and developers to **extract/inject KV cache** entries directly from/to vLLM’s internal **paged KV memory**. 
This paves the way for a new KV cache layer, and it's critical for workloads that depend on multi-user state sharing and fast user context switching across sessions and nodes.

---

## LMCache’s Two Core KV Cache Functions: Storage and Transport

LMCache now supports two major modes of operation:

### 1. **Storage: DB-Style Loading and Offloading**

This mode focuses on **persistent KV cache storage**. 
It’s perfect for workloads like **KV cache sharing** across user sessions or even between nodes in a cluster. 
LMCache maintains a database-style abstraction that can offload hot KV entries from GPU memory to CPU memory, or even to disk, using custom eviction and prefetch strategies. 
This enables long-term cache persistence and retrieval, dramatically increasing reuse rates for popular user prompts or conversation histories.

The new LMCache integration is designed to align with vLLM V1’s advanced memory features. It works seamlessly with **prefix caching**, **chunked prefill**, and the **paged memory manager**. But it doesn’t stop there. LMCache introduces a new set of APIs that extend the KV cache abstraction far beyond what was possible in v0.

These include:

- **Layer-by-layer KV injection**: Only need attention layers? You can inject them selectively.
- **Asynchronous KV extraction**: Extract KV entries without blocking the main inference thread.
- **KV cache prefetching**: Predict and load required KV chunks before the model even needs them.

Moreover, LMCache’s connector is now officially integrated into upstream vLLM, providing a plug-and-play experience for developers and researchers alike. No hacks. No forks. Just clean and efficient extensions to the vLLM core.

### 2. **Transport: P2P-Style Direct Transfer**

The second mode targets **disaggregated prefill** scenarios, where KV cache needs to be fetched from remote nodes in real-time. Instead of recomputing context tokens locally, LMCache enables **peer-to-peer KV transfers**, cutting down on redundant compute and unlocking cross-node cache reuse. This feature is crucial for multi-GPU, multi-node serving environments that demand extreme efficiency.

To push KV cache transport even further, LMCache now includes **first-class support for NIXL**, NVIDIA’s new communication abstraction designed for heterogeneous hardware. NIXL supports **NVLink**, **RDMA-capable NICs**, and **GPU Direct Storage**, offering unmatched bandwidth and latency across devices.

LMCache’s modular transport layer plugs directly into NIXL’s communication stack, allowing it to dynamically select the fastest available data path between GPUs, CPUs, and storage. Whether it’s a local multi-GPU system or a datacenter-scale cluster, LMCache with NIXL delivers blazing fast KV cache movement, often faster than recomputing even a single token’s attention.

This unlocks new LLM serving paradigms: real-time distributed KV cache retrieval, zero-compute context reconstruction, and even **memory disaggregation** at the hardware level.


---

## Benchmarks: LMCache vs. The State-of-the-Art

To understand the performance gain of the latest LMCache, we benchmarked LMCache with two leading vLLM-serving solutions **Standard K8s** and **Dynamo**. The following use cases reveal just how far LMCache pushes the boundaries of LLM inference.

We want to make two *important notes*: 
1. Note that since the baselines are not supported in vLLM V1 yet, our comparison with be based on vLLM V0.
2. Dynamo and AIBrix are constantly updating, and we will be happy to rerun these benchmarks once new versions are released. 

---

### **1. Disaggregated Prefill**

**Setup**: We tested these methods in a 1-Prefiller-1-Decoder setting. [Zhuohan, please fill in the details]

**Result**: [Insert figure]

**Takeaway**: Due to the use of NIXL, both LMCache and Dynamo enjoy fast direct transfer of KV caches between the "prefiller" GPU and the "decoder" GPU. As result, the performance of both system is much better than the standard K8s (without disaggregated prefill). 
Such performance gain aligns perfectly with next-gen data center architectures.

### **2. Time to First Token (TTFT) in Real World Traces**

**Setup**: We replayed the "ShareGPT" and "Mooncake" traces with the three systems, and speedup/slowdown the trace to simulate different workload intensity. Here, we assume that the system runs a disaggregated prefill architecture and only measure the time to first token (prefiller's delay).

**Result**: 

SharedGPT
[Insert figure]

Mooncake
[Insert figure]

**Takeaway**: As these traces include LLM input patterns in real-world applications, the results demonstrate that LMCache can improve performance in realistic workloads to various degrees. 


### **3. End-to-end performance with synthetic traces**

**Setup**: We simulate hundreds of users chatting simultaneously, each generating multiple turns in ShareGPT-style conversations. This stress-tests the system’s ability to cache, evict, and reuse conversation states on the fly.

**Result**: [Insert figure]

**Takeaway**: LMCache dramatically increases the effective size of the KV cache by intelligently offloading to CPU memory and selectively preloading GPU memory. As a result, reuse rates skyrocket, and the system becomes compute-bound earlier, avoiding cache misses and drastically reducing memory thrashing.

**Setup**: Users upload long documents as input (e.g., legal filings or research papers) and ask short questions about them. The input is large, but the output is minimal—classic retrieval-style interaction.

**Result**: [Insert figure]

**Takeaway**: In this scenario, LMCache’s fast inter-node KV cache transport—powered by our optimized NIXL kernel—enables **rapid context rehydration**. Even when working with multi-megabyte inputs, the system loads previously cached states faster than it would take to prefill them from scratch.


---

### **4. Retrieval-Augmented Generation (RAG)**

**Setup**: Each input document is chunked into segments (e.g., 512 tokens per chunk), with each query referencing multiple chunks. The RAG module uses embeddings to retrieve relevant chunks, and each chunk is injected as prefix to the LLM.

**Result**: [Insert figure]

**Takeaway**: LMCache’s **CacheBlend** module shines here. Unlike traditional setups that only cache the initial prompt prefix, CacheBlend enables **all retrieved chunks** to have their KV caches reused. This not only accelerates generation but also avoids repetitive prefill work when different queries hit overlapping content.

---

## The Road Ahead

LMCache Lab is just getting started. With vLLM V1 support and NIXL integration now shipping, we’re laying the foundation for a radically more efficient LLM serving stack—one that is disaggregated, distributed, and data-aware. KV cache is no longer just an optimization. It’s a **core abstraction** in the modern LLM runtime.

Stay tuned as we continue to push the boundaries of what’s possible in real-time, multi-user, multi-node language model inference.

---

Would you like help polishing visuals or embedding this into a Jekyll site post?
