---
title: "Computational Storage"
description: ""
lead: ""
draft: false
menu: 
  docs:
    parent: "apps"
weight: 2020
toc: true
---

## Basic Information

Computational storage device or **CSD** is a computer device which primary function is data storage, but that can also perform certain data processing functions. Functions can be either pre-configured or user-provided, both offline and online.

There are two main reasons one may need a CSD.

1. It can significantly reduce computational system design complexity by eliminating dependency from CPU and operating system. CSD may perform all essential storage functions like object space management, concurrency and transactions on-device. Accelerators may access CSDs directly over PCIe using database-like message-based protocols instead of transferring raw blocks of data (as in case of block storage devises).
1. It can significantly improve power outage and other types of failures handling by tightly coupling essential low-low level storage functions with device hardware. By controlling essential functions, device manufactures may _co-desing_ those functions and storage hardware.

Counter-intuitively, *speed* isn't considered here to be the main argument for using a CSD. Tight coupling of memory and compute will always give speed advantage. The problem is that storage media like NAND Flash doesn't like _heat_ produced by compute hardware. Putting massive radiators on devices will decrease storage density. As a result, intensive compute functions have to be disaggregated from CSDs into separate storage accelerators which are easier to cool.

Fortunately, one of the most important class of queries that benefit from using CDS, _point-like queries_, do not produce a lot of heat. 

Note that NAND Flash has pretty high access latency, that is much higher than PCIe latency. On the random object access CSD may show only a little performance improvements over a conventional block-based SSD.

## Simple Example: Persistent Queue

Queues are very important building block in event-driven distributed computing. Persistent queue saves incoming (pushed) messages on a durable media, removing them when they are being popped. In case of power failure, after restart system is restarted, may continue working because all in-flight messages are still in-light.

Normally, we need a CPU, a network device (NIC) and a block storage device like SSD. NAND Flash-based SSDs _may be_ a write bottleneck in this architecture because because flash has pretty high access latency -- 10th os microseconds. So we need to group and amortize writes in a _fast_ nvRAM buffer. 

But anyway, block devices operate via transferring _blocks_ and just to transfer 4K block over 4GB/s PCIe connection we need 1 microsecond, not including other latencies. And we probably need many of them to transfer _to and from_ RAM to commit a single `push()` operation.

{{< figure src="/docs/applications/storage/queue-bd.svg" >}}

If we want `push()` latency for a message to be within 5 microseconds over Ethernet, we need a optimize the whole data path. There are NIC-CPU latencies, including _OS kernel_ (that isn't that large, but nevertheless). 

The largest bottleneck is on the CPU-BlockDev side, because we need to transfer a lot of data, and each 4K block costs us 1 microsecond. PCIe latency on an ordinary (non CXL-optimized) platforms can easily be also about 1 microsecond. So each interaction with drive will be taking time. 

The obvious idea is to move the queue storage layer directly into the storage device, connected directly to the network device over a dedicated PCIe link. And let the device handle the queue completely. 

This wat we can save a lot of unnecessary block transfers and a lot of PCIe round-trips.

{{< figure src="/docs/applications/storage/queue-smart-stor.svg" >}}

Dedicated PCIe link can be pretty fast, with latency well under 1 microsecond, so we can safely assume that `push()` latency can be withing this number as well. Computational Network Device may handle networking part of the service, sending prepared data to the CSD. What is cool about this design that CPU is no more the bottleneck. Such queue may achieve very high message rates with extremely low latencies, bounded only be the network itself.

The main difference in this design is using message-oriented [HRPC](/docs/overview/hrpc) protocol instead of block-oriented NVMe to communicate with CSD. 

Latest generations of SSD controllers allow running Linux with applications, so running a persistent queue software or a _database_ software is not an issue. But Memoria can do _even better_.

## Execution Stack

Memoria has a pretty common stack of _essential_ abstractions for data-focused applications: processing layer ([DSLEngine](/docs/overview/vm)), data layer ([Containers](/docs/overview/containers)) and storage layer ([Storage engines](/docs/overview/storage)). Layers are perfectly isolated via virtual interfaces (can be compiled independently). HRPC-based interfaces provide binary compatibility.

The whole point of a CSD is to execute queries as close to the data as possible. Given fundamental power/heat constraints of memory media, it's not reasonable to run 'heavy' queries right on the device. Ideally, CSD should support a mode of operation/API when _raw data_ ca be exported to be processed on an external powerful accelerator.

Below is the diagram explaining it in greater details:

{{< figure src="/docs/applications/storage/io-stack.svg" >}}

Here the stack has two parts: HRPC Gateway (endpoints and services) and Memoria Essentials are open and provided by the Memoria Framework. Hardware-dependent part (low-level aspects of the Store's implementation and Hardware Abstraction Layer) may be open, but may be a closed vendor-specific hardware abstraction.

## Controller

Technically, we don't need a custom designed controller to run the execution stack. Any decent _64-bit_ SSD controller supporting embedded or external nvRAM should be enough. Custom chip can be equipped with accelerators so we can do _much more processing_ within CSD's allowed power budget. 

The idea is to use the same HW/SW architecture as outlined in [this document](/docs/overview/accel), adapted to specific requirements and constraints of computational storage devices. The controller consists of a cluster (grid, hypercube) of RISC-V based [processing elements](/docs/overview/accel#processing-element) (xPU) equipped with Memoria-specific ISA extensions accelerating algorithms and data structures. 

{{< figure src="/docs/applications/storage/accelerator.svg" >}}

Accelerator's architecture is 'standard' (unified). The only difference is problem-specific blocks (NAND Flash controllers) and, possibly, some other functions.

The main featurs of this controller's architecture are scalability and extensibility. We can add more processing elements if we want faster on-device query processing. Third-parties may add their own processing elements as long as they support HRPC interfaces of the accelerator.

One notable feature of this CSD architecture is that it does not need an [FTL](https://en.wikipedia.org/wiki/Flash_memory_controller) typical to block-based SSDs, that is usually provisioned at the rate of 1GB DRAM per 1TB NAND. All the memory in the system may be used for running queries.

## Storage Engines

Memoria currently has two storage engines for _block devices_ (BD) -- SWMRStore and OLTPStore, both supporting single serialized writer but multiple concurrent readers (SWMR), all in a wait-free mode. Both are transactional and support flexible commit policy. The biggest difference between them is that OLTPStore does not use block reference counters for memory management and is much faster for write transaction. But it does not support explicit version history because of that (there are other limitations too). BD-oriented storage engines need special _block allocator_. Notable feature of  SWMR storage engines in Memoria is that block allocator is transactional and has logarithmic worst-case allocation complexity. 

[NANDStore](/docs/overview/storage/#nandstore) is a version of OLTPStore, optimized for NAND Flash and [ZNS](https://zonedstorage.io/docs/introduction/zns) models of operation. The idea is that this storage engine is stacked on top of HRPC interfaces provided by the _Low Level Store/HAL_ of the CSD (see diagram above).

Note that SWMRStore can also be used inside a CSD, there are no technical restrictions for that. Moreover, it's preferable for analytical (read-optimized) applications that benefit the most from using CSDs. Just right now it's not a priority.

## Usage

There are three main features of Memoria-based CSDs:

1. Allows running _user-supplied queries_ in a secure, sandboxed environment on-device.
2. Much better failure recovery guarantees (including transactional durability), comparing to block-based storage devices like SSDs and HDDs, because CSD manufactures may control the entire _critical data path_.
3. We don't need a separate, dedicated CPU to use them.

Using CSD is straightforward. CSD provide HRPC message-based streaming interface, so any HRPC-enabled infrastructure may discover and use these resources as usual. From a functional perspective, working with CSD may look very much like working with a multimodel database via network connection.

Integration of CSDs as a storage devices into existing OS and applications is trickier. There is no technical issues in providing either block device interface (to run regular FS) on top of CSD, or to provide a regular FUSE/NFS-like remote interface to data as a virtual filesystem. 


