---
layout: post
title: "NVIDIA Dynamo integrates LMCache, Accelerating LLM Inference"
comments: true
thumbnail-img: "https://github.com/user-attachments/assets/84aa0337-6292-4f2c-aa3a-de12a0b61c22"
share-img: "https://github.com/user-attachments/assets/84aa0337-6292-4f2c-aa3a-de12a0b61c22"
author: NVIDIA Dynamo team, LMCache team
---

We're thrilled to announce that [**Nvidia Dynamo**](https://github.com/ai-dynamo/dynamo) **has integrated [LMCache](https://github.com/LMCache/LMCache) as a [KV caching layer solution](https://docs.nvidia.com/dynamo/latest/components/backends/vllm/LMCache_Integration.html)**. This is a big milestone: Dynamo gets a battle-tested caching solution, and LMCache becomes part of a data center-scale inference platform used by many developers worldwide to deploy AI at scale.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/84aa0337-6292-4f2c-aa3a-de12a0b61c22" />



For comprehensive details about Dynamo's KV cache optimization capabilities, see the **[NVIDIA Developer Blog post on reducing KV cache bottlenecks](https://developer.nvidia.com/blog/how-to-reduce-kv-cache-bottlenecks-with-nvidia-dynamo/)**.

## **Why KV Caching Matters**

KV caching is a foundational optimization for modern LLM inference. Instead of recomputing the expensive prefill phase for every new query, KV cache allows reuse of previously computed key/value pairs. This reuse **skips large portions of prefill computation**, dramatically reducing end-to-end latency while increasing throughput and efficiency.

We’ve explored this in detail in earlier posts, such as our [Turbocharging LLM Inference blog](https://blog.lmcache.ai/2025-05-16-release/), where we showed how KV cache reuse not only accelerates single-query latency but also enables more efficient multi-turn interactions and higher cluster utilization.

With Dynamo now supporting LMCache as a caching layer, these benefits become **first-class citizens** in the Dynamo platform.

## **What This Collaboration Delivers**

This collaboration focuses on two technical fronts:

### **1\. KV Cache Offloading and Reuse**

By default, KV cache is stored in GPU memory, which limits scale and context persistence. With this integration, Dynamo can now **offload KV cache to external storage layers** using LMCache while maintaining efficient reuse across queries. This integration is available on Dynamo repository: [ai-dynamo/dynamo\#2079](https://github.com/ai-dynamo/dynamo/pull/2079)

This combination enables scenarios like:

* Reusing KV cache across multiple sessions or even inference engines.  
* Freeing up GPU memory for active compute while keeping context cached externally.  
* Reducing prefill costs for long-context models by persisting and reloading KV segments.

### **2\. KV Cache Storage Backends**

Beyond offloading KV cache, Dynamo and LMCache now support flexible storage backends. For example, the [NiXL storage backend](https://github.com/LMCache/LMCache/blob/dev/lmcache/v1/storage_backend/nixl_storage_backend.py?utm_source=chatgpt.com) offers high-throughput, low-latency access optimized for LLM workloads. NIXL support is now available in LMCache repository: [LMCache/LMCache\#1223](https://github.com/LMCache/LMCache/pull/1223?utm_source=chatgpt.com)

This unlocks more advanced workflows:

* Persistent caches across application restarts.  
* Hybrid caching strategies (GPU memory \+ CPU memory \+ SSD) for balancing speed and cost.

### **Technical Reference**

For a deeper dive into the motivation, design scope, and integration details, see the official [Nvidia Dynamo documentation on LMCache integration](https://docs.nvidia.com/dynamo/latest/components/backends/vllm/LMCache_Integration.html?utm_source=chatgpt.com).

For more technical details about how Dynamo reduces KV cache bottlenecks and the broader context of this integration, check out the **[NVIDIA Developer Blog post on KV Cache optimization with Dynamo](https://developer.nvidia.com/blog/how-to-reduce-kv-cache-bottlenecks-with-nvidia-dynamo/)**.

## **Looking Ahead**

We’re excited to see how developers and enterprises adopt this integration in production. With KV caching becoming a standard practice across the industry, LMCache and Dynamo integration ensures that the ecosystem can move faster, serve more users, and deliver lower-latency AI applications.

Together, with the Dynamo team, we’re laying the foundation for a **more efficient, flexible, and cost-effective KV caching layer for LLM inference at scale**.

**Acknowledgements**

Special thanks to Vikram Mailthody, Harry Kim, Ashutosh Malegaonkar, Suman Taitraju, Richard Huo, Omri Kahalon, Vishwanath Venkatesan, Adit Ranadive, Pen Chung Li, John Kim, and David Edelsohn, in close collaboration with LMCache contributors from TensorMesh. 
