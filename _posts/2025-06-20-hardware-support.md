---
layout: post
title: "LLM Production Stack Goes Cross-Hardware: Ascend, Arm, and AMD Support Incoming"
thumbnail-img: /assets/img/lmcache-turbo/graph1.jpg
share-img:  /assets/img/lmcache-turbo/graph1.jpg
author: LMCache Team
image:  /assets/img/lmcache-turbo/graph1.jpg
---

**TL;DR:** Our [*LLM Production Stack*](https://github.com/vllm-project/production-stack) project just hit another milestone. We’re integrating with more hardware accelerators — including Ascend, Arm, and AMD — signaling growing maturity and broader applicability across enterprise and research settings.

---

### 🚀 LMCache Is Gaining Traction

LMCache has quietly become the unsung hero in the LLM inference world. As a core component of the *LLM Production Stack* project, it’s now the default KV cache layer in [llm-d](https://github.com/vllm-project/llm-d), supported by KServe, and increasingly adopted by enterprise users seeking scalable, performant LLM deployments.

But until now, LMCache has had a fairly tight relationship with NVIDIA. Much of its performance stems from CUDA-optimized kernels, making it ideal for NVIDIA GPUs — but limiting its portability across the broader hardware landscape.

That changes now.

---

### 🧠 Expanding Beyond CUDA

Recent pull requests (PRs) to LMCache have introduced support for new hardware platforms — **Ascend**, **Arm**, and **AMD**. These PRs are under active review, but once merged, they will unlock a new era for the LLM Production Stack: the ability to run across a range of heterogeneous accelerators.

This expansion marks a significant step forward in our mission to build a truly hardware-agnostic LLM inference stack. One that is production-grade, pluggable, and scalable across on-prem, cloud, and edge environments.

---

### 🔍 How the Integration Works

Let’s dig into the technical core.

At the heart of the LLM Production Stack is a crucial memory operation: **loading the KV cache from paged GPU memory (managed by vLLM) into CPU-resident chunked memory**. This operation balances latency and memory pressure, and is a performance-critical step in high-throughput LLM inference.

This step is implemented via a combination of:

* **PyTorch tensor operations**
* **Custom kernel optimizations** (originally CUDA-based)

Here’s the clever part: *only the kernel component is hardware-specific*. That means extending support to new platforms just requires rewriting or adapting the kernel logic for the relevant accelerator.

And that’s exactly what these new PRs do:

* One PR ports the kernel to **Ascend**’s custom instruction set.
* Another adapts it for **Arm-based CPUs**, optimizing cache movement across architectures.
* A third implements a HIP-compatible version for **AMD GPUs**, leveraging ROCm’s toolchain.

Each implementation preserves the original performance semantics while optimizing for the respective hardware.

---

### ⚙️ TPU Support? We’re on It.

And we’re not stopping there. We’re actively working with the Google Cloud team to bring **TPU** support to the LLM Production Stack. While this is still in early stages, the collaboration aims to enable TPU-based inference that benefits from the same KV caching optimizations and memory layout advantages.

Stay tuned — and if you’re working with non-NVIDIA hardware and want to get involved, we’d love your feedback and contributions.

---

### 🛠️ Building a Production Stack That Doesn’t Lock You In

The broader vision behind the *LLM Production Stack* is to provide a **modular**, **hardware-flexible**, and **high-performance** inference system for modern LLMs — whether you’re running models in the cloud, on your own servers, or at the edge.

With LMCache now going cross-hardware, that vision is closer than ever.

**→ Want to get involved or test out your hardware integration? [Join us on GitHub](https://github.com/vllm-project/production-stack) or drop into our [community Slack](https://vllm-dev.slack.com/archives/C089SMEAKRA).**

