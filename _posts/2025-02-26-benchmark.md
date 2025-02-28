---
layout: post
title: "ByteDance’s LLM serving stack is 10x SLOWER? Check out the real vLLM reference system!"
thumbnail-img: /assets/img/benchmark_e2e_brix.png
share-img: /assets/img/benchmark_e2e_brix.png
author: Production-Stack Team
image: /assets/img/benchmark_e2e_brix.png
---
<br>

## TL;DR

#### **A picture is worth a thousand words**

<div align="center">
<img src="/assets/img/benchmark_e2e_brix.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>

-  Widely thought to be TikTok/ByteDance’s internal system, **AIBrix** claims to have all the necessary features built out for production settings. However, this benchmark shows AIBrix is **way slower** and **less stable** than even the **naive Kubernetes** vLLM deployment!
-  Having heard similar complaints in the community after the initial hype around AIBrix, we release a **benchmark** study today for everyone to experiment and reproduce.

-  Now, ready a faster and deployment-ready option? Check out **vLLM Production Stack** [repo](https://github.com/vllm-project/production-stack), vLLM’s official reference implementation of a cluster-scale vLLM deployment with optimization (including **LMCache** [repo](https://github.com/LMCache/LMCache))!

- We release our **benchmark** [tutorial](https://github.com/vllm-project/production-stack/blob/main/tutorials/07-benchmark-multi-round-qa-multi-gpu.md) and [script](https://github.com/vllm-project/production-stack/tree/main/benchmarks/multi-round-qa) so you could try it yourself! 


##### [[vLLM Production Stack Repo]](https://github.com/vllm-project/production-stack) | [[More Tutorials]](https://github.com/vllm-project/production-stack/tree/main/tutorials) | [[Get In Touch]](https://forms.gle/Jaq2UUFjgvuedRPV8)

## Benchmark setting

*Workload:* This benchmark emulates a multi-turn conversation setting, where 320 users take turns to use the chat service hosted on a K8S cluster with 8 nodes each equipped with one A100 GPU running ```Llama-3.1-8B-Instruct```. 

We chose our conversation history from ShareGPT for real LLM input/output texts. Our workload contains 20,000 input tokens and 100 output tokens per user.

*Solutions:* 
- We set up [**vLLM Production Stack**](https://github.com/vllm-project/production-stack) by setting the CPU offloading buffer size (i.e., ``cpuOffloadingBufferSize``) to 120 GB per pod.

- For fair comparison, we also set up the **AIBrix** with the default setup and set "```VINEYARD_CACHE_CPU_MEM_LIMIT_GB```" to 120GB and ```cacheSpec memory``` to 150GB.

- As a reference point, we consider a simple K8S deployment with the same vLLM version and helm chart as used in the vLLM Production Stack, albeit without its KV cache and routing optimizations. We refer to it as **naive K8S**.



## Key results

### Faster response (time to first token)

First, we compare the average time-to-first-token (TTFT) delay as a function of queries per second (QPS) for these three solutions:

**AIBrix** (red line) shows a steep increase in TTFT as QPS rises (i.e., higher loads). This could be attributed to inefficient (pytorch-based) implementation of KV cache offloading and possibly other bottlenecks in other system components.

**Naive Kubernetes vLLM** (orange line) exhibits more stable performance but still experiences increasing latency with higher query rates due to lack of KV cache optimizations.

**vLLM Production Stack** (green line), with better KV cache optimization and router design, maintains consistently low TTFT across all QPS levels, demonstrating superior efficiency and scalability. The KV cache component is powered by [LMCache](https://github.com/LMCache/LMCache).


<div align="center">
<img src="/assets/img/benchmark_e2e_brix.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>



## Better availability

High availability is crucial in any production setting. 
We compare the stability of AIBrix and vLLM Production Stack over a period of serving time in the following experiment.

We run each deployment for a 400-second period.

<div align="center">
<img src="/assets/img/AIBrix_router_stability.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>
<div align="center">
<img src="/assets/img/prod_stack_stability.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>

The AIBrix router availability graph shows frequent downtime where the router becomes **unavailable** at **multiple** points during the test and requires **manually** restarting the port forwarding for the service to come back alive. 

On the other hand, **vLLM Production Stack** router availability graph (bottom) demonstrates **100% uptime** and with no interruptions throughout the same period. 



## Concluding Words

While benchmarking doesn't show the full picture, we are also surprised by the stark difference with this initial benchmarking. In short, AIBrix has a fully built-out native K8S integration, but vLLM Production Stack offers **better performance** and **availability**, and from-the-scratch **modular** design.

Born out of an academic **collaboration** between **vLLM** (Berkeley) and **LMCache** (UChicago), vLLM Production Stack features the most advanced built-in **KV-cache optimizations** and an upstream support of the latest vLLM releases.

As an **open** framework, vLLM Production Stack uses helm and python interface for ease of use and modification. As a near-term priority, we welcome **contributions** from the community to add more K8S native support, including **CR-based deployment**, and LoRA management, among others. 

We also welcome **more benchmarks** on different workloads and inclusion of other serving frameworks. Contact us in the **#production-stack** [channel](https://vllm-dev.slack.com/archives/C089SMEAKRA) or LMCache slack today to discuss the future steps!

Join us to build a future where every application can harness the power of LLM inference—reliably, at scale, and without breaking a sweat.

*Happy deploying!*

## Contacts:

- **Github: [https://github.com/vllm-project/production-stack](https://github.com/vllm-project/production-stack)**
- **Chat with the Developers** **[Interest Form](https://forms.gle/mQfQDUXbKfp2St1z7)**
- **vLLM Production-Stack [channel](https://vllm-dev.slack.com/archives/C089SMEAKRA)**
- **LMCache [slack](https://join.slack.com/t/lmcacheworkspace/shared_invite/zt-2viziwhue-5Amprc9k5hcIdXT7XevTaQ)**
