---
layout: post
title: "AIBrix vs Production-Stack, which one should I use for vLLM K8S Serving?"
thumbnail-img: /assets/img/benchmark_brix.png
share-img: /assets/img/benchmark_brix.png
author: Production-Stack Team
image: /assets/img/benchmark_brix.png
---
<br>


## TL;DR
- vLLM boasts the largest open-source community in LLM serving. Inside the vLLM project, **“AIBrix”** is an solution open-sourced by ByteDance team, while **“vLLM production-stack”** offers a light-weight reference solution which focuses on integration with vLLM engine.
- Despite the implementation difference and origin, how do **AIBrix** and **Production Stack** really compare in terms of **performance**, **stability**, and **functionality**?
- We release our benchmark on this [[Link]] so you could try it yourself. Let us know if you are interested in connecting!

##### [[Github Link]](https://github.com/vllm-project/production-stack) | [[More Tutorials]](https://github.com/vllm-project/production-stack/tree/main/tutorials) | [[Get In Touch]](https://forms.gle/Jaq2UUFjgvuedRPV8)


# The Context
<!-- Over the past year, LLM inference has raced to the forefront, powering everything from chatbots to code assistants and beyond. It’s quickly becoming critical infrastructure, much like the cloud was to big data, cellular was to mobile apps, and CDNs were (and still are!) to the broader Internet. -->


**vLLM** has taken the open-source community by storm, with unparalleled hardware and model support plus an active ecosystem of top-notch contributors. 

**vLLM Production-stack** is an open-source **reference implementation** of an **inference stack** built on top of vLLM, designed to run seamlessly on a cluster of GPU nodes. 

**AIBrix** is an open-source solution created by ByteDance developed separately with similar functionalities as their production scenario.

Today, we release an open-benchmark for community to compare in terms of **performance**, **stability**, and **functionality**.


# Benchmark Results
We report our results based on three aspects: Performance, Stability, and Functionality.

## Performance

End to End comparison graph

## Availability

XXX

## Functionality
Table comparison, AIBrix has lora,xxx



## Conclusion

AIBrix and Production-Stack are both great open-source vLLM cluster serving solutions. While AIBrix gives a K8S-native solution with Lora support, Production-Stack offers better performance and stability due to its light-weight design and close co-design with vLLM serving engine.

We welcome more benchmarks on different workloads and inclusion of other serving frameworks. Talk with us in the #production-stack channel in vLLM slack or LMCache slack today to discuss the future steps!

Join us to build a future where every application can harness the power of LLM inference—reliably, at scale, and without breaking a sweat.

*Happy deploying!*

Contacts:
- **Github: [https://github.com/vllm-project/production-stack](https://github.com/vllm-project/production-stack)**
- **Chat with the Developers** **[Interest Form](https://forms.gle/mQfQDUXbKfp2St1z7)**
- **vLLM [slack](https://slack.vllm.ai/)**
- **LMCache [slack](https://join.slack.com/t/lmcacheworkspace/shared_invite/zt-2viziwhue-5Amprc9k5hcIdXT7XevTaQ)**
