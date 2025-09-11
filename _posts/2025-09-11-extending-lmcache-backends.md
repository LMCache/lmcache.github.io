---
layout: post
title: "Extending LMCache Backends: A Comprehensive Guide to Custom Backend Development"
subtitle: "Learn how to build custom backends for LMCache using the external backend extension mechanism"
tags: [backend, extension, customization, storage, lmcache]
comments: true
author: Baolong, Kobe
---

## Abstract

In large language model inference scenarios, the performance and flexibility of KVCache caching systems directly impact overall service efficiency. LMCache, as a high-performance large model caching framework, provides developers with rich extension capabilities through its modular backend design. This article will start with LMCache backend's extension mechanism, using the officially provided `lmc_external_log_backend` as an example, to detail how to customize backends that meet business requirements, including design principles, implementation details, and practical steps. We hope that various vendors can easily extend their own backends based on this guide.

## LMCache's Existing Backends

<div align="center">
<img src="/assets/img/lmcache_backend.png" alt="LMCache Backend Overview" style="width: 90%; vertical-align:middle;">
<p><em>LMCache Backend Architecture Overview</em></p>
</div>

As shown in the diagram above, LMCache currently provides numerous backends that serve as storage and transmission adapters for KVCache.

Among them, `LocalCpuBackend` is the most commonly used backend and is essential in most cases, as it not only serves as a hot cache but also takes responsibility for allocating the necessary MemoryObjects for storage and retrieval processes.

Another special one is `NixlBackend`, whose purpose is to transmit KVCache to peer nodes rather than sending it to local or remote storage.

In addition, `RemoteBackend` is currently the most easily extensible backend, adopting the Connector framework, which allows storage providers to integrate with LMCache more conveniently.

Finally, the highlighted `ExternalBackend` is the dynamically extensible external backend that this article will introduce in detail.

## Why Extend LMCache Backends?

LMCache provides common backends based on memory, local disk, GDS, etc., by default. However, in actual production environments, developers may face the following customization needs:

- **Special storage medium adaptation**: Need to integrate with distributed storage (such as Ceph), non-volatile memory (NVM), and other special media
- **Customized persistence strategies**: Need to implement specific persistence logic such as write-ahead logging (WAL) or snapshot-based approaches to ensure data reliability
- **Business-level monitoring and auditing**: Need to embed business logic such as log reporting and performance statistics into cache operations
- **Compatibility and compliance**: Need to adapt to internal storage protocols or meet requirements for data encryption, access control, and other compliance needs

`lmc_external_log_backend` can be viewed as an extension example designed for "logging" scenarios. It provides an excellent reference for understanding the extension mechanism by implementing LMCache's abstract interface and recording cache operations in log format.

## Core Mechanism of LMCache Backend Extension

LMCache adopts an "abstract interface + concrete implementation" design pattern, where all backends must follow unified interface specifications to ensure decoupling from the framework's core logic. Before starting development, you need to master the following core concepts:

### Core Abstract Interface Definition

LMCache defines the general interface in `StorageBackendInterface` in `lmcache/v1/storage_backend/abstract_backend.py`.

<div align="center">
<img src="/assets/img/StorageBackendInterface.png" alt="StorageBackendInterface Structure" style="width: 85%; vertical-align:middle;">
<p><em>StorageBackendInterface Class Structure</em></p>
</div>

The core methods that extended backends need to implement include:

| Interface Method | Function Description |
|------------------|---------------------|
| `__init__(self, config, metadata, loop, memory_allocator, local_cpu_backend: LocalCPUBackend, dst_device, lookup_server=None)` | Initialize the backend (such as opening files, connecting to storage). The constructor should refer to the latest definition in LMCache's `lmcache/v1/storage_backend/__init__.py` in the `create_dynamic_backends` method to maintain consistency |
| `contains(self, key: CacheEngineKey, pin: bool = False) -> bool` | Check if the specified key exists |
| `exists_in_put_tasks(self, key: CacheEngineKey) -> bool` | Check if the key is in asynchronous put tasks |
| `batched_submit_put_task(self, keys: Sequence[CacheEngineKey], objs: List[MemoryObj], transfer_spec=None)` | Submit a batch of put tasks at once |
| `submit_prefetch_task(self, key: CacheEngineKey) -> bool` | Submit prefetch tasks |
| `get_blocking(self, key: CacheEngineKey) -> Optional[MemoryObj]` | Get key content in blocking mode |
| `get_non_blocking(self, key: CacheEngineKey) -> Optional[Future]` | Get key content asynchronously, returning a Future object for waiting for data content |
| `batched_get_blocking(self, keys: List[CacheEngineKey]) -> List[Optional[MemoryObj]]` | Get a batch of key contents in blocking mode at once |
| `pin(self, key: CacheEngineKey) -> bool` | Pin the MemoryObject corresponding to this key to prevent eviction |
| `unpin(self, key: CacheEngineKey) -> bool` | Unpin this key so it can be evicted |
| `remove(self, key: CacheEngineKey, force: bool = True) -> bool` | Delete this key |
| `batched_remove(self, keys: list[CacheEngineKey], force: bool = True) -> int` | Delete a batch of keys at once, returning the actual number of deleted objects |

Any custom backend only needs to inherit the `StorageBackendInterface` abstract class and implement the above interfaces to seamlessly integrate into LMCache.

### Process for Using Extended Backends

LMCache loads custom backends through the following process, achieving decoupling through "develop extension - install extension - configure extension":

1. **Develop Extension**: Inherit `StorageBackendInterface`, reference `lmc_external_log_backend`, implement interface functions, then package into a wheel package
2. **Install Extension**: Install the extension locally via `pip install <backend whl>`
3. **Configure Extension**: Users specify `external_backends: "<new extension backend>"` in the LMCache configuration file, such as specifying `log_external_backend` as the self-extended backend. You can also provide comma-separated values to specify multiple external backends

Additionally, you need to configure `extra_config` to specify additional configuration for custom external extensions.

Here's a configuration example using `log_external_backend`:

```yaml
chunk_size: 64
local_cpu: False
max_local_cpu_size: 5
external_backends: "log_external_backend"
extra_config:
  external_backend.log_external_backend.module_path: lmc_external_log_backend.lmc_external_log_backend
  external_backend.log_external_backend.class_name: ExternalLogBackend
```

## Custom External Backend Example: lmc_external_log_backend Introduction

`lmc_external_log_backend` is an official backend example provided by LMCache. Its function is the simplest implementation of necessary interfaces that prints logs. Its core goal is to demonstrate the backend extension approach.

<div align="center">
<img src="/assets/img/ExternalLogBackend.png" alt="ExternalLogBackend Implementation" style="width: 80%; vertical-align:middle;">
<p><em>ExternalLogBackend Class Implementation Structure</em></p>
</div>

As shown in the diagram above, `ExternalLogBackend` inherits from LMCache's `StorageBackendInterface` and implements the agreed-upon `__init__` method and other abstract methods.

### 1. Backend Initialization

```python
class ExternalLogBackend(StorageBackendInterface):
    def __init__(
        self,
        config,
        metadata,
        loop,
        memory_allocator,
        local_cpu_backend: LocalCPUBackend,
        dst_device,
        lookup_server=None,
    ):
        super().__init__(dst_device=dst_device)
        self.config = config
        self.metadata = metadata
        self.loop = loop
        self.memory_allocator = memory_allocator
        self.lookup_server = lookup_server
        logger.info(f"ExternalLogBackend initialized for device {dst_device}")
```

### 2. Implementing Core Methods

Here are a few example methods; for complete implementation, please refer to the code repository:

```python
def contains(self, key: CacheEngineKey, pin: bool = False) -> bool:
    logger.info(f"ExternalLogBackend Checking contains for key: {key}")
    return False

def batched_submit_put_task(
    self,
    keys: List[CacheEngineKey],
    objs: List[MemoryObj],
    transfer_spec=None,
) -> Optional[List[Future]]:
    for key, obj in zip(keys, objs):
        # Record storage operation for each key
        logger.info(f"ExternalLogBackend Put task for key: {key}, size: {obj.tensor.size()}")
    return None

def submit_prefetch_task(self, key: CacheEngineKey) -> bool:
    logger.info(f"ExternalLogBackend Prefetch task for key: {key}")
    return True
```

### 3. Complete Implementation Reference

This log backend doesn't actually store data but records all operations to the logging system. For complete code, please refer to the GitHub repository [lmc_external_log_backend](https://github.com/opendataio/lmc_external_log_backend). The code repository explains how to package, create wheel packages, and install them, which this article won't elaborate on.

## Summary

LMCache's external backend extension mechanism provides developers with powerful customization capabilities:

1. **Flexible Integration**: Supports various storage systems
2. **Non-intrusive**: No need to modify existing LMCache repository code
3. **Dynamic Loading**: New backends can be enabled through dependency installation and configuration only
4. **Easy Development**: Clear interfaces and documentation support

Whether you need to add monitoring, support new hardware, or implement custom storage strategies, extending LMCache backends can efficiently achieve these goals. The `lmc_external_log_backend` project is an excellent starting point that can help you quickly understand the backend development process.
