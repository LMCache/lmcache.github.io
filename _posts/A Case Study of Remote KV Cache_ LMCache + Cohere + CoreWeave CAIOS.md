---
layout: post
title: "Breaking the Memory Barrier: How LMCache and CoreWeave Power Efficient LLM Inference for Cohere"
thumbnail-img: /assets/img/async.png
share-img: /assets/img/async.png
author: Walter Beller-Morales (Cohere), Samuel Shen (Tensormesh), Kishor Aher (CoreWeave)
image: /assets/img/async.png
---

# **Breaking the Memory Barrier: How LMCache and CoreWeave Power Efficient LLM Inference for Cohere**

By Walter Beller-Morales (Cohere), Samuel Shen (Tensormesh), Kishor Aher (CoreWeave)

### **The challenge: Scaling enterprise AI**

Enterprises today are racing to integrate large language models (LLMs) into their products and workflows, but doing it at scale brings challenges in performance, cost, and accuracy. Organizations need models to be based on their specific data, while making sure that this information remains private. [**Cohere**](https://cohere.com), one of the leading enterprise AI companies, built its North platform to help organizations use their own internal data safely and effectively to power retrieval-augmented generation (RAG). North allows enterprises to ground model outputs in trusted, private knowledge bases, delivering accurate, contextual responses tailored to their business.

When you use RAG, you are prefixing each request with the relevant contextual data so that the model will provide relevant answers. This approach introduces a computational hurdle as it adds large amounts of data which need to re-processed everytime a query is received as this information does not modify the weights of the model but only stores it in the KV cache which is a temporary memory which is typically discarded after the query is processed. The richer the context an LLM is given, the more **tokens** it must process, and behind every few tokens lies a growing [**Key and Value tensors**](https://medium.com/analytics-vidhya/understanding-q-k-v-in-transformer-self-attention-9a5eddaa5960) **(KV) cache** that stores intermediate model states. This cache is essential for generating coherent responses, but it grows rapidly with input length, consuming vast amounts of GPU or CPU memory.

At scale, this creates a performance and cost bottleneck for inference. Even for efficient inference engines like vLLM. Cohere’s engineering team set out to solve this problem by exploring whether **KV caches could be stored remotely**, freeing up local memory without slowing down inference.

That’s where **LMCache** and **CoreWeave AI Object Storage** come in. Together, they enable high-performance **remote KV caching**, allowing large models to handle long contexts with less memory pressure and better throughput.

In this blog, we’ll examine recent Cohere benchmark tests which demonstrate how these technologies come together to **power some of the world’s most complex AI workloads**, achieving remarkable efficiency and scalability in real-world inference.

### **Remote KV caching**

To address the growing memory demands of large-scale inference, **LMCache** reimagines how language models manage and store their context. At the heart of every transformer-decoder based LLM lies the **Key and Value tensors (KV) cache**, the hidden state data that must be preserved across tokens so the model can maintain coherence in its output. As input length increases, this cache grows rapidly: every few tokens produce additional KV pairs that accumulate over time. The result is a steep rise in memory usage that quickly becomes a bottleneck, even for advanced inference platforms.

LMCache solves this by implementing a remote KV cache architecture. Instead of keeping all cache data in GPU or CPU memory, LMCache serializes and stores it externally, retrieving it only when needed. This approach significantly reduces memory pressure on inference hardware, allowing for longer contexts, more simultaneous sessions, and better resource utilization, without sacrificing, and even improving, model performance.

To make remote caching viable, storage must be both high-throughput and latency-tolerant. **CoreWeave AI Object Storage** provides exactly that foundation. Designed for AI workloads, CoreWeave AI Object Storage delivers multi-gigabyte-per-second bandwidth and resilient scalability across GPU clusters. It ensures that KV data can be offloaded, persisted, and fetched back at the speed modern inference demands.

Together, LMCache and CoreWeave AI Object Storage form a tightly integrated system: LMCache handles cache serialization and coordination, while CoreWeave AI Object Storage provides the distributed performance backbone that makes external caching seamless. The result is a new, more flexible model of inference, one where context can grow without constraint and infrastructure can scale intelligently.

### **Benchmark testing**

To evaluate how well remote caching performs under real-world workloads, the LMCache team partnered with **Cohere** to benchmark its integration on the North platform, using CoreWeave AI Object Storage as the storage backend. The goal was to determine whether inference could remain fast and efficient even when the KV cache lives outside of GPU or CPU memory.

The tests were conducted on [Cohere’s Command A model](https://cohere.com/blog/command-a), running on CoreWeave’s GPU infrastructure with the vLLM inference engine. Three configurations were compared:

1. **Baseline:** Full prefill, with no KV cache reuse.

2. **LMCache \+ CoreWeave AI Object Storage:** KV data serialized and stored on CoreWeave AI Object Storage, then retrieved on demand.

3. **Alternative Object Storage (S3 Express):** Used as a performance baseline for cold and hot cache conditions.

The benchmarks measured two key metrics:

* **Time to First Token (TTFT):** How quickly the model generates its first token — dominated by the prefill phase.

* **Decoding Throughput:** The number of tokens generated per second once prefill is complete.

<div align="center">
<img src="/assets/img/TTFT.png" alt="Icon" style="width:400px; vertical-align:middle;"><img src="/assets/img/batchdecode.png" alt="Icon" style="width:400px; vertical-align:middle;">
</div>

The results were decisive:

* **22–32% lower TTFT**, meaning faster model response times.

* **41% higher decoding throughput** compared to full prefills.

* **1.2× performance improvement** over S3 Express for cold cache retrievals.

* **3× faster performance** than S3 Express when the cache was already hot.

These results show that with the right architecture, remote caching can outperform traditional memory-bound inference, dramatically reducing hardware strain, and help control costs.

**Why it works: The technology behind the power**

On paper, moving a model’s cache to remote storage should slow everything down. After all, storage bandwidth is an order of magnitude lower than GPU or CPU memory access speeds. But in practice, LMCache and CoreWeave AI Object Storage turn this assumption on its head, combining architectural innovations that make remote caching not just viable, but performant.

### **1\. Asynchronous KV cache loading**

Traditional inference systems treat the KV cache as a blocking dependency: when the model requests cache data, GPU computation halts until it’s available. LMCache replaces this with a non-blocking, asynchronous loading mechanism.

When a request is made, LMCache begins streaming the cache from storage while the model continues decoding tokens already in progress. If the requested cache isn’t ready, LMCache signals the inference engine to “check back later.” The GPU keeps working, and by the time it does check back, the cache is typically ready to slot in.

This overlap between computation and I/O eliminates idle GPU time, effectively masking storage latency under active compute and keeping inference throughput high.

<div align="center">
<img src="/assets/img/async.png" alt="Icon" style="width:400px; vertical-align:middle;">
</div>


### **2\. High-throughput remote storage** 

The second enabler is CoreWeave’s infrastructure. CoreWeave AI Object Storage delivers 8.5–10 GB/s of throughput per 8-GPU node, enough to sustain rapid cache transfers even across large model workloads. While that’s lower than direct CPU-GPU I/O, the combination of high throughput and LMCache’s async architecture ensures that storage operations happen invisibly in the background.

The result: a seamless integration where remote bandwidth bottlenecks are hidden under GPU computation, allowing the system to behave as though all the cache were local.

Together, these two innovations redefine how LLMs can scale. Instead of being limited by memory size or local I/O speed, inference can now expand dynamically with performance shaped by architecture, not hardware limits.

## **A new paradigm for scalable inference**

The collaboration between Cohere, LMCache, and CoreWeave highlights a pivotal shift in how large language models are optimized for enterprise-scale performance. For years, improving inference meant adding more GPUs, expanding clusters, or fine-tuning batching strategies. But as context lengths increase and workloads become more dynamic, a new question emerges: *how can we make inference smarter, not just faster?*

For Cohere, which powers mission-critical enterprise AI through its North platform, the answer lies in rethinking how memory is managed. By pairing LMCache’s remote KV caching technology with CoreWeave’s high-throughput infrastructure, Cohere demonstrated that it’s possible to maintain — and even improve — inference speed while reducing memory overhead. The result is a more scalable, cost-efficient, and flexible architecture for real-world applications.

This partnership shows that the KV cache (long treated as a low-level implementation detail) can instead become a strategic asset. When stored and reused intelligently, it enables:

* Persistent sessions that resume instantly, even after clusters are scaled down.

* Cross-node scalability that allows cache data to flow seamlessly between inference systems.

* Lower operational costs without sacrificing model performance or responsiveness.

For enterprises deploying models at scale, these capabilities are transformative. The work done by Cohere, LMCache, and CoreWeave signals the arrival of a new inference paradigm, one where efficiency comes from architectural intelligence rather than raw compute power.

As LLMs continue to evolve, this collaboration serves as a proof point for what’s possible: by integrating cutting-edge caching systems with high-performance cloud infrastructure, the world’s most advanced AI platforms can deliver faster, leaner, and more sustainable performance.

Learn more about:

* [Cohere North](https://cohere.com/north)  
* [CoreWeave](https://coreweave.com/why-coreweave)  
* [Tensormesh](http://www.tensormesh.ai)

