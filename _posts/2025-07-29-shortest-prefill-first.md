---
layout: post
title: "LMCache Lab: Shortest Prefill First—Smarter Scheduling for Lightning-Fast Prefill Nodes!"
thumbnail-img: /assets/img/spf_blog/overall.png
share-img: /assets/img/spf_blog/overall.png
author: Kuntai Du
image: /assets/img/spf_blog/overall.png
---

***TL;DR:*** ⚡ **Shortest Prefill First (SPF)** scheduling cuts LLM time-to-first-token by up to **18%** in prefill-decode disaggregation!

---

At LMCache Lab, we are obsessed with LLM performance. In this blog, we introduce **Shortest Prefill First (SPF)**: a smarter scheduling technique that accelerates your prefill nodes by executing requests with shortest prefill time first. This reduces the average waiting time for all requests, leading to much lower average time-to-first-token (TTFT). Also, it optimizes your KV cache locality because those requests with longer cache hits will typically have lower prefill time.

## Why Shortest Prefill First? 🏃‍♂️

In large LLM serving clusters—especially those using prefill-decode separation—prefill nodes can get slowed down due to request queueing. With **SPF** (Shortest Prefill First), we always schedule the requests with shortest prefill time first. This reduces the waiting time of all requests in queue, allowing your customer to see the requests sooner.

Further, this technique also optimizes for KV cache locality, as those requests with longer cache hit will be prioritized (as they typically have lower prefill time than others), which means that, when this technique combined with a KV cache backend (e.g. LMCache), it can yield more saving to your users.

In short, we summarize that SPF can:
    - **Reduce average TTFT by 18%:** asdf
    - **Ideal for prefill-decode disaggregation:** Prefill nodes become dramatically more responsive.
    - **Supercharges full-stack LLM serving:** When paired with a KV cache management system (like LMCache), SPF can make scheduling and cache reuse much smarter.

## How Does It Stack Up? 📊

Here’s a head-to-head of *mean* and *median* time-to-first-token (TTFT) for different LLM serving platforms (all with output length = 1):

| Approach                       | Mean TTFT (ms) | Median TTFT (ms) |
|--------------------------------|---------------:|-----------------:|
| **vLLM (native)**              |     7,500.75   |      6,280.66    |
| **vLLM (SPF)**                 |     6,116.30   |      4,816.27    |
| **Fireworks**                  |     8,052.40   |      7,943.03    |
| **DeepInfra**                  |     7,664.74   |      6,564.00    |

**Result:**  
Shortest Prefill First delivers an **18% reduction** in mean TTFT vs. native vLLM, and even more when compared to Fireworks and DeepInfra. Median TTFT shows similar gains—SPF consistently pulls the fastest response out of your hardware!

## Why Does SPF Shine with LMCache? 🌟

A key insight:  
**SPF’s impact multiplies in a system with full KV cache awareness.** LMCache tracks request history and cache reuse, letting SPF *always* prioritize requests that can finish and cache fastest. This synergy powers LMIgnite, allowing it to:

- **Outperform LLM-d, Dynamo, and vLLM-native NIXL connector**
- **Dramatically reduce tail latencies for bursty or uniform-length workloads**
- **Keep your GPU utilization sky-high with no wasted time**

## How to Try It 🛠️

Want to play with SPF right now?  
- **Code:** [GitHub: SPF branch](https://github.com/TensorMesh-Internal/vllm/tree/kuntai/shortest_job_first)
- **Best results:** Set output length = 1, and ensure input lengths are uniform.
- **Sample command:**
  ```bash
  python benchmark_serving.py --model meta-llama/Llama-3.1-8B-Instruct --dataset-name random --random-input-len 10000 --random-output-len 1 --random-range-ratio 0.9 --request-rate inf --num-prompts 50
  ```

## Pro tip:
When combined with LMCache, SPF is fully cache-aware, letting you skip redundant prefill work and accelerate your entire pipeline.

## LMIgnite: The Fast Lane 🚦

Want these speedups out-of-the-box? **LMIgnite** will bring SPF, KV cache-aware scheduling, and more to your cloud or cluster - no engineering required on your side! **Sign up** [here](https://lmignite.tensormesh.ai/) for early access.

Faster prefill, faster decode, more efficient infra—the LMCache Lab way! 🚀
