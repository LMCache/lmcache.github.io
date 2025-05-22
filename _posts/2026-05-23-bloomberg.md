---
layout: post
title: "Bloomberg Cuts LLM Inference Cost Using LMCache, Open-Source KV Cache Layer for vLLM"
thumbnail-img: https://github.com/user-attachments/assets/2ea85a52-f0fe-44fd-80c7-e04896911e60
share-img:  https://github.com/user-attachments/assets/2ea85a52-f0fe-44fd-80c7-e04896911e60
author: LMCache Team
image:  https://github.com/user-attachments/assets/2ea85a52-f0fe-44fd-80c7-e04896911e60
---

Bloomberg, a global leader in financial technology and data, has successfully deployed [LMCache](https://github.com/LMCache/LMCache), an open-source KV cache layer optimized for LLM inference, into its production systems. Over the past three months, Bloomberg has used LMCache to significantly reduce the cost and latency of running large language models (LLMs) at scale.

<img width="600" alt="image" src="https://github.com/user-attachments/assets/51cc311e-1338-410c-9ba6-b2a0f0a3292a" />

As financial institutions increasingly turn to LLMs to process sensitive documents, real-time news, and structured financial data, privacy and control have become paramount. Public LLM APIs, while convenient, often fall short of internal compliance and data protection standards. For this reason, enterprises like Bloomberg are embracing open-source LLMs hosted within their own secure environments.

Yet with this shift comes a new challenge: inference cost. Running open-source LLMs in-house requires substantial GPU resources, and the cost scales quickly with usage. That’s where LMCache comes in.

Over the past eight months, the LMCache development team and Bloomberg’s machine learning infrastructure group have worked closely to integrate and optimize LMCache for Bloomberg’s needs.

> *(Insert quote placeholder from Bloomberg engineer or infrastructure lead about collaboration and responsiveness)*

LMCache’s high-performance design yields a cache hit rate of 40–50% in Bloomberg’s environment. This translates to fewer redundant GPU operations and faster responses for end-users—especially in workloads involving repeated queries or conversational history. Much of the gain comes from LMCache’s advanced CPU offloading techniques, which minimize contention on GPU compute cycles.

Deploying LMCache was seamless. Bloomberg integrated it into their infrastructure by simply switching the Docker image used to serve models via vLLM, an increasingly popular LLM inference engine.

> “LMCache has been a game-changer for us,” said Vadim Markovtsev, an engineering manager at Bloomberg. “It allowed us to cut inference costs meaningfully without compromising latency or reliability. The fact that it's open-source and so easy to integrate made it an obvious choice.”

Beyond raw performance, LMCache includes enterprise-ready features that Bloomberg relies on, such as KV cache pinning (to persist key-value pairs across sessions) and detailed cache hit/miss metrics for internal observability.

Looking ahead, Bloomberg and the LMCache community are exploring deeper integrations, including support for retrieval-augmented generation (RAG), agentic workflows, and more advanced multi-model deployments.

For more information about LMCache, visit: [https://github.com/LMCache/LMCache](https://github.com/LMCache/LMCache)

