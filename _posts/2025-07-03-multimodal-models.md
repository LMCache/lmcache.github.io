---
layout: post
title: "LMCache Extends Its Turbo-Boost to Multimodal Models in vLLM V1"
thumbnail-img: /assets/img/mm.png
share-img:  /assets/img/mm.png
author: LMCache Team
image:  /assets/img/mm.png
---

**Take-away:** The latest LMCache release plugs seamlessly into vLLM's new multimodal stack. By hashing image-side tokens (`mm_hashes`) and caching their key-value (KV) pairs, LMCache reuses vision embeddings across requests—slashing *time-to-first-token* and GPU memory for image-LLMs.

<div align="center">
<img src="/assets/img/mm.png" alt="LMCache Multimodal Architecture" style="width: 87%; vertical-align:middle;">
<p><em>LMCache's multimodal caching architecture</em></p>
</div>

## Summary — Why This Matters

Multimodal large language models (MLLMs) multiply KV-cache traffic: every image can add thousands of "vision tokens." Without reuse, identical pictures force the GPU to re-encode vision features for each query.
LMCache 0.3.1 [PR \#882](https://github.com/LMCache/lmcache/pull/882) adds a hash-based lookup so repeated images hit the cache instead of the GPU pipeline. In a demo with Qwen-VL-2B:


| Query | Total tokens | KV hits | Hit rate |
| :-- | :-- | :-- | :-- |
| First image | 16 178 | 0 | 0% |
| Same image (2 nd) | 16 178 | 16 177 | ~100% |
| New image (3 rd) | 4 669 | 0 | 0% |

The second request streamed in ~1 s vs. ~18 s cold-start, showing near-instant reuse.


## 1 | Introduction — How vLLM Handles Multimodality

vLLM V1 exposes a `multi_modal_data` field; vision inputs are tokenised by a **vision encoder → projector** and spliced into the text sequence as *placeholder ranges*.
For each image it also emits a **16-byte hex digest (`mm_hash`)** that uniquely identifies the visual content.

## 2 | The Bottleneck Before LMCache

1. **Linear KV growth.** Every pixel-token sits in GPU memory for the whole session—dominating long chats.
2. **Bandwidth bound.** Attention kernels stall on DRAM when moving these extra KV blocks.
3. **No cross-request reuse.** Identical images in consecutive requests pay the full vision-encoder cost again.

## 3 | How LMCache Enables Multimodal Caching

The merged patch adds three key pieces:

- `apply_mm_hashes_to_token_ids()` overwrites dummy vision-token IDs with a 16-bit hash (cheap Torch slice).
- `RequestTracker` now stores `mm_hashes` + positions, so every KV lookup/insert sees the true hash.
- On cache store/retrieve, LMCache treats those hashed IDs like normal tokens—no special path needed.

As a result, **identical images map to identical token sequences**, and KV reuse "just works" in both storage and P2P transport modes.

## 4 | Performance Gains in Practice

Early integration tests on Qwen-VL-2B and DeepSeek-VL2-Small show:

- ~100% hit rate for repeated images (table above) → *zero* GPU vision passes.
- End-to-end latency trimmed from tens of seconds to ≈1 s for second views.
- KV bandwidth drops proportionally, freeing GPU memory for larger batches—on text-only workloads LMCache already yields **3-10× TTFT speed-ups**; the same economics now apply to vision prompts.


## 5 | Simple Demo

```bash
# 1. launch vLLM with LMCache connector
vllm serve Qwen/Qwen2-VL-2B-Instruct \
  --kv-transfer-config '{"kv_connector":"LMCacheConnectorV1","kv_role":"kv_both"}' \
  --no-enable-prefix-caching --enforce-eager

# 2. POST an image prompt twice – watch the logs
curl -X POST http://localhost:8000/v1/chat/completions -H "Content-Type: application/json" \
  -d @first_image.json
curl -X POST http://localhost:8000/v1/chat/completions -H "Content-Type: application/json" \
  -d @first_image.json   # instant response, log shows ~16k KV hits
```

Replace the JSON with any Hugging Face MLLM prompt syntax; identical images will hit the cache automatically.

## Join the LMCache Ecosystem
LMCache is rapidly expanding its integration footprint.

We're actively seeking partnerships with LLM inference frameworks and cloud providers interested in:

- KV cache optimization
- Distributed inference pipelines
- High-efficiency on-prem or VPC-based deployments


Let's make AI infrastructure faster, cheaper, and smarter—together.

Try LMCache: https://github.com/LMCache/LMCache
Follow us on [Linkedin](https://www.linkedin.com/company/lmcache-lab/?viewAsMember=true), [Twitter](https://x.com/lmcache) 