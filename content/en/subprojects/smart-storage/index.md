---
title: "Smart Storage"
description: ""
lead: ""
date: 2021-10-28T02:06:54-04:00
lastmod: 2021-10-28T02:06:54-04:00
draft: false
images: []
menu: 
  subprojects:
    parent: "subprojects"
weight: 10
toc: true
---

One of the most complex problems in memory architectures is how to handle loss of power. For applications working with persistent memory, this problem is reducing to the procedure of atomic dumping of the state from volatile memory to a non-volatile memory. Even if the main memory is non-volatile, after power is restored, the entire system is in unknown state. Because power loss may occur in-between a long series of no-atomic memory updates. To successfully survive a loss of power, even NVRAM must be updated in a series of persistent atomic and recoverable steps.

Unfortunately, atomic operations are pretty expensive and hard to implement. Persistent storage block devices like HDDs and SSDs support them in a limited way: only a few consecutive blocks can be updated atomically. Filesystems and database storage engines may utilize this functionality for better resiliency, but it doesn't make much difference, usually. It's really a hard technical challenge to dump generic application's state from non-volatile memory to persistent memory in a fault-tolerant way.

There is some activity in this direction. Database engines support write-ahead logging (WAL), some filesystems utilize various copy-on-write (CoW) techniques. Zoned block devices (ZBD) provide WAL-like functionality at the device level. And, if it's supported in software (filesystems and databases), can be used to to provide much more resilient and performant storage for applications than traditional block devices.

The main idea here is to split [SWMRStore](/docs/dbfs/swmr-store) into separate layers, and implement either its CoW block layer, or even the entire store in a storage device's controller. Something like ZBD, but instead on WAL it will be based on CoW.

Feature-wise, SWMRStore (pronounced "swimmer-store") is somewhere between databases and filesystems. It provides linearizable multi-statement transactions (strong serializability) on top of virtually any [data structure](/docs/overview/data-structure), at the same time maintaining rather low level API and high IO efficiency of filesystems. So, applications may have best of both worlds.

Internally, SWMRStore is pretty similar to [LMDB](http://www.lmdb.tech/doc/), except the following things:

1. SWMRStore supports more block sizes: 4K, 8K, 16K, ... 1M. 
1. SWMRStore supports checkpoints, so applications can opt-in into relaxed durability gaining some speed. 
1. SWMRStore provides history of transactions and branches (all branches are still single-writer though).
1. SWMRStore does not depend on memory mapping and may operate on top of any block device using high-performance IO interfaces like IOCP/Linux AIO/[io_uring](https://en.wikipedia.org/wiki/Io_uring).

LMDB is build on top of memory-mapping, with rather simple data model (key/value data only), it's really lightning-fast because of that, and has a small code footprint. SWMRStore is not nearly as lightweight as LMDB (because of tons of additional features and better scalability), but it's still the same provably-correct wait-free [MVCC](https://en.wikipedia.org/wiki/Multiversion_concurrency_control), because of Copy-on-Wright scheme.

It's not that everything is good in single-writer/multiple-readers (SWMR) CoW kingdom. This storage management scheme has two main limitations:

* **Only single read-write transaction at a time is possible**. This is a fundamental limitation that can be only slightly relaxed in some cases. Concurrent writers, if they are required, have to be somehow dispatched using the single writer transaction, sacrificing isolation and making rollbacks much more expensive.
* **CoW requires ether some garbage collection (GC) or reference counting (RC) to reclaim the space**. GC is seriously limiting the scalability of CoW-based storage, and RC is hard to implement in a copy-on-write way efficiently. So, RC itself is usually not transactional. 

SWMRStore is using RC-based space reclamation scheme, and in case of unexpected power loss, partial storage scan (typically, 1-10% of *allocated* blocks) is required to rebuild block counters *before* the storage is writable again (counters are not needed for reading). Though it's expected that recovery time is rather small in absolute numbers (like, a few seconds), but, nevertheless, recovery has $O(M)$ time complexity, where $M$ is the data volume. In case when it's not acceptable, GC is an option.

Another limitation is that block reference counters are operated in RAM, so for RC-based CoW we do need a RAM buffer of the size, linearly proportional to the size of data.

But, given that those limitations are not vital for an application, SWMR CoW can theoretically achieve very high transaction rates, up to millions TPS per consistency domain (usually a database or a filesystem, or a shard). This is because CoW is intrinsically wait-free in the data path, and there is only one (no transaction history) or two locks (with transaction history and branches) in the store, which, technically, can be implemented directly in hardware for ultimate efficiency. Other than those one or two locks, readers and writers do not interact with each other in any way when accessing the data.

Another important property of SWMR CoW is that live data is never over-written between transactions. In other words, if some physical block $B$ was written in transaction $T_1$, it will never be overwritten in transaction $T_i, i>1$, unless it's freed in some $T_j, j > 1, j < i$. Notable exceptions are superblocks, but there is a fixed number of them, so they can be handled specially. Blocks within a writable transaction can be over-written (updated), but both reader and writer transactions in Memoria are intrinsically single-threaded. So, such updates are naturally serialized anyway.

So, in SWMRStore:
* concurrent multi-threaded access is only possible to immutable (read-only) blocks; 
* mutable blocks are only accessed sequentially;
* and there is a clear transactionally-aligned block life-cycle (allocate-update-free).

**Specifics of this block access pattern can be used for both simplifying the data stack and making it more resilient and fault-tolerant, comparing with traditional block devices, which are designed for the more general case: concurrent access to the mutable data**.

For example, NAND flash SSDs have pretty complex controller inside, implementing specific copy-on-write techniques over NAND flash chips. This is because NAND can't naturally handle block-sized data updates. Having CoW over NAND, then Block Device API over this CoW, then SWMRStore over this BD API looks like adding a lot of unnecessary complexity into the data stack. It's logical to allow SWMRStore to manage NAND flash directly, **skipping generic blocks device API emulation completely**. Technically, SWMRStore has it's own block ID translation layer, that can be extended to handle flash directly. What it the most important, manufacturers of such SWMRStore-based block devices can deeply optimize everything together, keeping critical inventions in secret by not exposing them publicly via device API.




