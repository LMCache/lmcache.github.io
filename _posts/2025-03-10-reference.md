---
layout: post
title: "vLLM Production Stack: a necessary step to AGI"
thumbnail-img: /assets/img/benchmark_e2e_brix.png
share-img: /assets/img/benchmark_e2e_brix.png
author: Production-Stack Team
image: /assets/img/benchmark_e2e_brix.png
---
<be>





## TL;DR

- vLLM Production Stack, originally an academic open-source project, is now an official reference implementation of a cluster-wide full-stack vLLM serving system, including monitoring, CR-based router, autoscaling, LoRA management, distributed KV sharing, etc.
- Why does it matter to the industry? 
  - A growing open community with official upstream vLLM support.
  - Designed for both optimized performance and being Kubernetes-native.


## The origin

AGI doesn't just mean better models--it also means better systems. Each organization that takes AGI seriously needs the best LLM serving infra, high availability, scalability, speed, and low cost.

vLLM Production Stack is borne out of a long academic collaboration between the vLLM team (UC Berkeley) and the LMCache team (UChicago), a KV-cache optimization system ([CacheGen](https://dl.acm.org/doi/10.1145/3651890.3672274), [CacheBlend](https://arxiv.org/abs/2405.16444)). 
As more vLLM and LMCache users ask for help to deployment these projects in production, we see a pressing need for an official reference implementation of a cluster-wide vLLM serving system. Thus, we released the vLLM Production Stack project in early January 2025. At this point, it also supports native Kubernetes deployment, autoscaling, and LoRA management. 



## Open community + Official vLLM support

The project welcomes **EVERYONE** to contribute. It has a growing contributor community with over 30 active contributors from various companies worldwide, including IBM, Lambda, ??, ??
Check out our community meeting notes here: [ASK YUHAN FOR LINKS]

At the same time, we ensure that the latest vLLM Production Stack is always compatible with the latest vLLM releases. We achieve this by merging the LMCache Connector in the upstream main-branch vLLM. [LINK TO MERGE]

In short, as our community grows, people never need to worry whether their contributions in the production stack might conflict with the vLLM releases.



## K8S-native deployment + Optimized performance

When deploying Kubernetes-native solutions, operators often face the difficult choice between "K8S-native" and "high performance." With vLLM Production Stack, you don't have to choose. 

It supports [CR-based router](link), [sidecar-based LoRA adapter management](link), [K8S-native autoscaling](link), and [distributed KV cache sharing](link) (direct point-to-point sharing or hierarchical structure). 

In short, vLLM production stack showcases the promise when academic research and industry expertise join forces! 


## Concluding Words

**We also welcome more benchmarks on different workloads and other serving frameworks**. Contact us in the **#production-stack** [channel](https://vllm-dev.slack.com/archives/C089SMEAKRA) or LMCache slack today to discuss the future steps!

**Join us to build a future** where every application can harness the power of LLM inference—reliably, at scale, and without breaking a sweat.

*Happy deploying!*

## Contacts:

- **Github: [https://github.com/vllm-project/production-stack](https://github.com/vllm-project/production-stack)**
- **Chat with the Developers** **[Interest Form](https://forms.gle/mQfQDUXbKfp2St1z7)**
- **vLLM Production-Stack [channel](https://vllm-dev.slack.com/archives/C089SMEAKRA)**
- **LMCache [slack](https://join.slack.com/t/lmcacheworkspace/shared_invite/zt-2viziwhue-5Amprc9k5hcIdXT7XevTaQ)**
