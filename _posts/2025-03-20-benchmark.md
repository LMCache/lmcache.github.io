---
layout: post
title: "Open-Source LLM Inference Cluster Performing 10x FASTER than this Big Tech Solution? Check out the vLLM Production-Stack!"
thumbnail-img: /assets/img/benchmark_e2e_brix.png
share-img: /assets/img/benchmark_e2e_brix.png
author: Production-Stack Team
image: /assets/img/benchmark_e2e_brix.png
---
<br>

## TL;DR

<div align="center">
<img src="/assets/img/benchmark_e2e_brix.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>

- [**vLLM Production Stack**](https://github.com/vllm-project/production-stack), an official [**open-source reference implementation**](https://docs.vllm.ai/en/latest/deployment/k8s.html) of a cluster-wide, full-stack vLLM serving system, was first released in Jan 2025 by researchers from vLLM and UChicago. Since then, the system has gained popularity and have been tested under **production** scenarios. Current features include K8s-native router, autoscaling, CR-based LoRA management, distributed KV sharing, monitoring, etc.

- In the mean time, **AIBrix**, which was released from **ByteDance** recently, also claims to have all the necessary features built out for production settings. 

-  Today, we release a **benchmark** to demonstrate that **vLLM Production Stack** could have **10X better** performance than AIBrix in a multi-turn chat application besides being more **stable**. Moreover, we show that **AIBrix** perform even **worse** than a **naive vLLM** + K8s setup.

- In order to make it easy for everyone to reproduce the results and test with more benchmarks, we relase our [scripts](https://github.com/vllm-project/production-stack/blob/main/tutorials/07-benchmark-multi-round-qa-multi-gpu.md) and [tutorial](https://github.com/vllm-project/production-stack/blob/main/tutorials/07-benchmark-multi-round-qa-multi-gpu.md) to further facilitate the development of open-source LLM serving solutions.

##### [[vLLM Production Stack Github]](https://github.com/vllm-project/production-stack) | [[Get In Touch]](https://forms.gle/Jaq2UUFjgvuedRPV8) | [[Slack]](https://vllm-dev.slack.com/archives/C089SMEAKRA) | [[Linkedin]](https://www.linkedin.com/company/lmcache-lab) | [[Twitter]](https://x.com/lmcache)

## Benchmark Results

### Solutions to test today: 

- [**vLLM Production Stack**](https://github.com/vllm-project/production-stack): We use the default settings of vLLM Production Stack (v0.1.0) and set the CPU offloading buffer size (i.e., ``cpuOffloadingBufferSize``) to 120 GB per pod.

- **AIBrix**: For fair comparison, we used the stable release of v0.2.0 with the default setup and set "```VINEYARD_CACHE_CPU_MEM_LIMIT_GB```" to 120GB and ```cacheSpec memory``` to 150GB. (120GB per pod)

-  **Naive K8S** As a reference point, we consider a simple K8S deployment with the same vLLM version and helm chart as used in the vLLM Production Stack, albeit without its KV cache and routing optimizations.

*Workload:* This benchmark emulates a multi-turn conversation setting, where 320 users take turns to use the chat service hosted on a K8S cluster with 8 nodes each equipped with one A100 GPU running ```Llama-3.1-8B-Instruct```. 

We chose our conversation history from ShareGPT for real LLM input/output texts. Our workload contains 20,000 input tokens and 100 output tokens per user.



### Results
First, we compare the average time-to-first-token (TTFT) delay as a function of queries per second (QPS) for these three solutions:

**AIBrix** (red line) shows a steep increase in TTFT as QPS rises (i.e., higher loads). This could be attributed to inefficient (pytorch-based) implementation of KV cache offloading and possibly other bottlenecks in other system components.

**Naive Kubernetes vLLM** (orange line) exhibits more stable performance but still experiences increasing latency with higher query rates due to lack of KV cache optimizations.

**vLLM Production Stack** (green line), with better KV cache optimization and router design, maintains consistently low TTFT across all QPS levels, demonstrating superior efficiency and scalability. The KV cache component is powered by [LMCache](https://github.com/LMCache/LMCache), with efficient KV transfer CUDA kernels and dedicated zero-copy design.


<div align="center">
<img src="/assets/img/benchmark_e2e_brix.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>


Regarding the inter-token-latency (ITL), we observe a similar pattern that while the ITL of both AIBrix and Naive K8s went up, production stack remains constantly low latency. 
<div align="center">
<img src="/assets/img/benchmark_e2e_2.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>

## Better availability

High availability is crucial in any production setting. For this reason, We compare the stability of AIBrix and vLLM Production Stack over a period of serving time in the following experiment.

We run each deployment for a 400-second period.

<div align="center">
<img src="/assets/img/AIBrix_router_stability.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>
<div align="center">
<img src="/assets/img/prod_stack_stability.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>

The AIBrix router availability graph shows frequent downtime where the router becomes **unavailable** at **multiple** points during the test and requires **manually** restarting the port forwarding for the service to come back alive. 

On the other hand, **vLLM Production Stack** router availability graph (bottom) demonstrates **100% uptime** and with no interruptions throughout the same period. 

### More Experiments
We will show more experiment results in this section. In the first set of experiments, we change the number of rounds that user have chat with into 40 rounds per session. Improvement of vLLM Production Stack grows even higher due to the increasing chance of KV cache reuse.

 <div align="center">
<img src="/assets/img/benchmark_e2e_3.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>

<div align="center">
<img src="/assets/img/benchmark_e2e_4.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>

We also tested our performance on bigger models such as Llama-70B. In the following graphs.We emulated running a cluster with 4 nodes, each running Llama-3.1-70B-Instruct with tensor parallelism on 2 A100 GPUs with 80GB. The other specifications per pod stay the same.

We assumed a time period of 70 users participating in 20 rounds of conversations with the LLM. Each requests consists of 9,000 input tokens per user and outputs 10 tokens.

<div align="center">
<img src="/assets/img/benchmark_e2e_5.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>

<div align="center">
<img src="/assets/img/benchmark_e2e_6.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>

## Concluding Words

While benchmarking doesn't show the full picture, we hope this blog shows that **open-source** is not necessarily worse than the **industry** solution, but could also be **10X better**.

Born out of an **academic collaboration** between **vLLM** (Berkeley) and **LMCache Lab** (UChicago), vLLM Production Stack features the most advanced built-in **KV-cache optimizations** and an upstream support of the latest vLLM releases.

As an **OPEN** framework, vLLM Production Stack uses helm and python interface for ease of use and modification. As a near-term priority, **we welcome contributions from the community to add more K8S native support, including Envoy Endpoint, dynamic Lora Pool, among others**. 

**We also welcome more benchmarks on different workloads and other serving frameworks**. Contact us in the **#production-stack** [channel](https://vllm-dev.slack.com/archives/C089SMEAKRA) or LMCache [slack](https://join.slack.com/t/lmcacheworkspace/shared_invite/zt-2viziwhue-5Amprc9k5hcIdXT7XevTaQ) today to discuss the future steps!

**Join us to build a future** where every application can harness the power of LLM inference—reliably, at scale, and without breaking a sweat.

*Happy deploying!*

## Contacts:

- **Github: [https://github.com/vllm-project/production-stack](https://github.com/vllm-project/production-stack)**
- **Chat with the Developers** **[Interest Form](https://forms.gle/mQfQDUXbKfp2St1z7)**
- **vLLM Production-Stack [channel](https://vllm-dev.slack.com/archives/C089SMEAKRA)**
- **LMCache [slack](https://join.slack.com/t/lmcacheworkspace/shared_invite/zt-2viziwhue-5Amprc9k5hcIdXT7XevTaQ)**
