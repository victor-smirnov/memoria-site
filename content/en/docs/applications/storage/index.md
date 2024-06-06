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

## The IO Stack

{{< figure src="/docs/applications/storage/io-stack.svg" >}}

## Controller

{{< figure src="/docs/applications/storage/accelerator.svg" >}}

## SWMRStore



## 


