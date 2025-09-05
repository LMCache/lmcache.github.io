---
layout: post
title: "Nvidia Dynamo + LMCache: Accelerating the Future of LLM Inference"
thumbnail-img: /assets/img/lmcache_nvidia.png
share-img: /assets/img/lmcache_nvidia.png
tags: [nvidia, dynamo, performance, distributed-inference, lmcache, collaboration]
comments: true
author: Junchen, Kobe, Jiayi, Samual
image: /assets/img/lmcache_nvidia.png
---

We're thrilled to announce that the [Nvidia Dynamo](https://github.com/ai-dynamo/dynamo) project has integrated [LMCache](https://github.com/LMCache/LMCache) as its [KV caching layer solution](https://docs.nvidia.com/dynamo/latest/components/backends/vllm/LMCache_Integration.html). This is a big milestone: Dynamo gets a battle-tested caching solution, and LMCache becomes part of a production-scale ecosystem used by many developers worldwide.

![LMCache NVIDIA Integration](/assets/img/lmcache_nvidia.png)
*LMCache and NVIDIA Dynamo: A powerful combination for distributed LLM inference*

## Why KV Caching Matters

KV caching is a foundational optimization for modern LLM inference. Instead of recomputing the expensive prefill phase for every new query, KV cache allows reuse of previously computed key/value pairs. This reuse skips large portions of prefill computation, dramatically reducing end-to-end latency while increasing throughput.

We've explored this in detail in earlier posts, such as our [May 2025 release blog](https://blog.lmcache.ai/2025-05-16-release/?utm_source=chatgpt.com), where we showed how KV cache reuse not only accelerates single-query latency but also enables more efficient multi-turn interactions and higher cluster utilization.

With Dynamo now adopting LMCache as its caching layer, these benefits become first-class citizens in the Dynamo ecosystem.

## What This Collaboration Delivers

This collaboration focuses on two technical fronts:

### 1. KV Cache Offloading and Reuse

By default, Dynamo stores KV cache in GPU memory, which limits scale and persistence. With LMCache integration, Dynamo can now offload KV cache to external storage layers while maintaining efficient reuse across queries. Dynamo's implementation PR: [ai-dynamo/dynamo#2079](https://github.com/ai-dynamo/dynamo/pull/2079)

This combination enables scenarios like:
- Reusing KV cache across multiple sessions or even inference engines.
- Freeing up GPU memory for active compute while keeping context cached externally.
- Reducing prefill costs for long-context models by persisting and reloading KV segments.

### 2. KV Cache Storage Backends

Beyond offloading, Dynamo and LMCache now support flexible storage backends. For example, the [NiXL storage backend](https://github.com/LMCache/LMCache/blob/dev/lmcache/v1/storage_backend/nixl_storage_backend.py?utm_source=chatgpt.com) offers high-throughput, low-latency access optimized for LLM workloads. LMCache's support PR: [LMCache/LMCache#1223](https://github.com/LMCache/LMCache/pull/1223?utm_source=chatgpt.com)

This unlocks more advanced workflows:
- Persistent caches across application restarts.
- Hybrid caching strategies (GPU + CPU + SSD) for balancing speed and cost.

## Technical Reference

For a deeper dive into the motivation, design scope, and integration details, see the official [Nvidia Dynamo documentation on LMCache integration](https://docs.nvidia.com/dynamo/latest/components/backends/vllm/LMCache_Integration.html?utm_source=chatgpt.com).

## Looking Ahead

These efforts are led by the Dynamo team including [ZichengMa](https://github.com/ZichengMa), [Richard Huo](https://github.com/richardhuo-nv), [Ziqi Fan](https://github.com/ziqifan617), [Tomer Shmilovich](https://github.com/tshmilnvidia), in close collaboration with LMCache contributors. Together, we're laying the foundation for a more efficient, flexible, and cost-effective KV caching layer for LLM inference at scale.

We're excited to see how developers and enterprises adopt this integration in production. With KV caching becoming a standard practice across the industry, LMCache + Dynamo ensures that the ecosystem can move faster, serve more users, and deliver lower-latency AI applications.
