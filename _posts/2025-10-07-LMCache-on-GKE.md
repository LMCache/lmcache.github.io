---
layout: post
title: "LMCache on Google Kubernetes Engine: Boosting LLM Inference Performance with KV Cache on Tiered Storage"
comments: true
thumbnail-img: /assets/img/lmcache-gke-tiered-storage.png
share-img: /assets/img/lmcache-gke-tiered-storage.png
image: /assets/img/lmcache-gke-tiered-storage.png
author: Danna Wang, Google
---
## **Overview of the Collaboration**

The KV Cache is a memory optimization that makes Large Language Models(LLMs) run the forward pass faster by storing Key (K) and Value (V) matrices to prevent the model from recalculating them for the entire text sequence with every new generated token. Maximizing the KV Cache hit rate with storage is paramount for delivering high-performance LLM inference. The KV Cache's immense GPU memory footprint, amplified by system prompt size, is the primary bottleneck in LLM serving, directly constraining context length, concurrency, and overall system throughput. Our primary goal is to maximize the KV Cache hit ratio by effectively expanding GPU HBM with a tiered node-local storage solution. To achieve this, our collaboration with the LMCache team (Kuntai Du, Jiayi Yao, Yihua Cheng who all work at Tensormesh, the company that they formed after collaborating on LMCache) leverages their intelligent software to create this solution on GKE.

### **Tiered Storage Approach** 

LMCache allows extending the KV Cache from the GPU's fast HBM(Tier 1) to larger, cost-effective tiers like CPU RAM and local SSDs. This innovative approach dramatically increases the total cache size, which provides a better chance of maximizing the hit ratio and improves overall inference performance, since more data is stored and available locally on the accelerator node. For GKE users, this means accommodating models with massive context windows while maintaining excellent performance.
Performance Benchmarking and Results

We designed our tests to measure the performance of a tiered KV Cache by configuring workloads with total cache sizes that would fill each storage layer (HBM, CPU RAM, Local SSD). These configurations were benchmarked using various context lengths (1k, 5k, 10k, 50k, and 100k tokens) representing multiple use cases , with TTFT, token input throughput and end-to-end latency as our primary performance indicators.  Some examples of use cases for different system prompt length:
 * 1k - 5k: High-Fidelity Personas and Complex Instructions
 * 10k: Average user prompt in tokens (small RAG) or  tokens in a webpage / article
 * 50k: Prompt Stuffing
 * 100k: Tokens in a long book

The results below capture the best-performing storage setup for each KV Cache size and the improvements it provides.

## **Experiment Setup**
We deployed a vLLM server on a A3 mega machine, utilizing local SSD for ephemeral storage (through emptyDir). The detailed test setup is provided below:
 * Hardware: 8 × nvidia-h100-mega-80gb GPUs
 * Model: Llama-3.3-70B-Instruct
 * LMCache version: v0.3.3 
 * Cache Configuration
    * HBM only
    * HBM + CPU RAM 
    * HBM + CPU RAM + Local SSD 
 * Storage resource: HBM: 640Gi, CPU RAM: 1Ti, Local SSD 5Ti
 * Benchmark tool: SGLang bench_serving
 * Requests: The test was conducted using several system prompt lengths (1k, 5k, 10k, 50k, and 100k tokens). Each system prompt provided a shared context for a batch of 20 inference requests. While the system prompt was shared, each individual request consisted of a unique 256-token input and generated a 512-token output.

### Example command
```bash
python3 sglang/bench_serving.py --host=${IP} --port=${PORT} --dataset-name='generated-shared-prefix' --model=$MODEL --tokenizer=$MODEL --backend=vllm --gsp-num-groups=80 --gsp-prompts-per-group=20 --gsp-system-prompt-len=1000 --gsp-question-len=256 --gsp-output-len=512 --request-rate=800 --max-concurrency=200
```

## **Benchmark results**
We create our test with different total KV Cache sizes. The below results capture the best storage setup for each KV Cache sizes and the improvement they make:

**Test 1:** The total cache of 1.1M \- 1.3M tokens fit entirely within HBM.  
**Results:**  In this scenario, adding slower storage tiers offered no advantage, making an HBM-only configuration the optimal setup.

**Test 2:** The cache size was increased to 4.0M \- 4.3M tokens, exceeding HBM's capacity but smaller than HBM \+ CPU RAM capacity.  
**Results:** 

| System Prompt Length | Best-performing Storage Setup  | Mean TTFT (ms) Change (%) comparing with HBM only setup | Input Throughput Change (%) comparing with HBM only setup | Mean End-to-End Latency Change (%) comparing with HBM only setup |
| :---- | :---- | :---- | :---- | :---- |
| 1000 | HBM | 0% | 0% | 0% |
| 5000 | HBM \+ CPU RAM | \-18% | \+16% | \-14% |
| 10000 | HBM \+ CPU RAM | \-44% | \+50% | \-33% |
| 50000 | HBM \+ CPU RAM \+ Local SSD | \-68% | \+179% | \-64% |
| 100000 | HBM \+ CPU RAM \+ Local SSD | \-79% | \+264% | \-73% |

**Test 3:** With a large cache of 12.6M \- 13.7M tokens, the workload saturated both HBM and CPU RAM, spilling over to the Local SSD. The total size of cache is smaller than HBM \+ CPU RAM \+ Local SSD capacity.  
**Results:** 

| System Prompt Length | Best-performing Storage Setup  | Mean TTFT (ms) Change(%) comparing with HBM only setup | Input Throughput Change (%) comparing with HBM only setup | Mean End-to-End Latency Change (%) comparing with HBM only setup |
| :---- | :---- | :---- | :---- | :---- |
| 1000 | HBM \+ CPU RAM | \+5% | \+1% | \-1% |
| 5000 | HBM \+ CPU RAM | \-6% | \+27% | \-21% |
| 10000 | HBM \+ CPU RAM \+ Local SSD | \+121% | \+23% | \-19% |
| 50000 | HBM \+ CPU RAM \+ Local SSD | \+48% | \+69% | \-41% |
| 100000 | HBM \+ CPU RAM \+ Local SSD | \-3% | \+130% | \-57% |

## Summary and Next Steps

As you can see from the results above the tiered storage solution improves inference performance by using node-local storages, particularly in scenarios with long system prompts that generate large KV Caches. 

Improving LLM inference performance is a complex challenge where multiple infrastructure components (storage, compute, networking) must work together. Our efforts are part of a larger program to optimize the entire end-to-end inference stack—from intelligent load balancing at the [Inference Gateway](https://cloud.google.com/kubernetes-engine/docs/concepts/about-gke-inference-gateway) to advanced caching logic in the model server.

We are continuing to explore further enhancements by integrating additional remote storage solutions with LMCache. 

![LMCache on GKE Tiered Storage](/assets/img/lmcache-gke-tiered-storage.png)

**Next Steps:**

* To get started you can try the same setup [mentioned above on GKE](https://github.com/vllm-project/production-stack/blob/main/tutorials/cloud_deployments/04-GCP-GKE-lmcache-local-disk.md).  
* [Keep up to date on the LLM-D Inference Stack](https://llm-d.ai/) 

