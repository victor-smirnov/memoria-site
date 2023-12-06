---
title: "Storage Engines"
description: ""
lead: ""
draft: false
images: []
menu:
  docs:
    parent: "overview"
weight: 40
toc: true
---

Memoria has pluggable storage engines. Block storage API is isolated behind the [IStore](https://github.com/victor-smirnov/memoria/blob/master/containers-api/include/memoria/core/container/store.hpp#L132) interface. Block life-cycle in the code is [managed](https://github.com/victor-smirnov/memoria/blob/master/containers-api/include/memoria/profiles/common/block_cow.hpp#L30) with RAII. As far as some block is used (directly or *indirectly* referenced) in the code, a storage engines knows about that and my, for example, keep it in a cache.

There are four 'core' storage engines in Memoria, covering basic usage: in-memory, disk-based OLTP-optimized, analytics-optimized and 'static immutable files'. The may be many other, for secondary purposes, like, embedding into the executable image, and so on.

**NOTE THAT (1) all storage engines are currently in the experimental state, features declared here may be missing or incomplete, and (2) no any on-disk data format stability is _currently_ implied. Physical data layout may and will be changing from version to version.**

## Persistent Data Structures ##

A data structure is **persistent** if update operations (a single one or a batch of) produce new version of data structure. Old version may be kept around or removed as soon as they are not in use. Read access to old versions does not interfere with write access creating new versions. So very few if any synchronization between threads is required. Persistent data structures are mostly wait-free, that makes them a good fit for concurrent environments (both distributed and multi-core single-machine).

Virtually every dynamic data structure can be made persistent, but not every one can be made persistent *natively* the way that creating a new version does not result in copying of entire data structure. For example, there are efficient copy-on-write (CoW) based algorithms for persistent trees without parent links (see below). But not for trees with parent or sibling links, linked lists or graphs. Fortunately, there is a way to "stack" non-persistent data structures on top of a persistent tree, and as a result, the former gains persistent properties of the latter by the expense of an *O(log N)* extra memory accesses. Memoria follows this way. 

### Persistent Trees ###

Persistent balanced tree is very much like ordinary (non-persistent) tree, except instead of updating tree in-place, we create a new tree that shares the most with "parent" tree. For example, updating a leaf results in copying the entire path form the root to the leaf into a new tree: 

{{< figure src="cow-tree.svg" >}}

Here, balanced tree for Version 1 consists from yellow path and from the rest of Version's 0 tree *excluding yellow path*. Version's 2 tree consists from blue path and the rest of Version's 1 tree *excluding blue path*. For insertions and deletions of leafs the idea is the same: we are copying modified path to a new tree and referencing the the rest on the old tree.

### Atomic Commitment ###

Updates never touch old versions (CoW). Many update operations can be combined into a single version, effectively forming an atomic *snapshot*. 

{{< figure src="cow-memory-p.svg" >}}


If writers buffer a series of updates in a thread-local memory and, on update completion, publishes those updates to shared memory atomically, then we have **snapshot atomic commitment**. Other readers will see either finished and consistent new version of the data or nothing. So, if a version fits into local memory of a writer thread and we never delete versions, we have [Grow-Only Set CRDT](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type#G-Set_(Grow-only_Set)), that works well without explicit synchronization.

## Transactional Operations ##

Note that versions in fully-persistent data structures are *not* transactions, despite providing perfect isolation and atomicity. Each update (or a group of updates) create a new *version* of data, and to have transactions we must be able either:

1. Select one version out of many at the transaction start time, or
2. Merge multiple versions into a single one at the commit time.

But *within a single line of history*, versions are essentially transactions.

### Reference Counting and Garbage Collection ###

Deletion of versions is special. To delete a version means to delete every persistent tree's node that is not referenced in other versions, down to the leaves:

{{< figure src="cow-tree-delete.svg" >}}

Here we are deleting Version's 0 root node, as well as green and blue nodes, because they are overshadowed by corresponding paths in other versions. To determine which node is still referenced in other versions, we have to remember *reference counter* for each node. On the deletion of a version, we recursively decrement nodes' reference counters and delete them physically, once counters reach the zero.

In case of deletions, persistent tree's nodes are still a [2P-Set CRDT](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type#2P-Set_(Two-Phase_Set)), so insertions and deletions do not require synchronization. Unfortunately, reference counting requires linearizable memory model for counters. Memory reclamation in persistent trees does not scales well because of linerizability requirements for reference counting. But in practice it may or may not necessary be a problem in a concurrent environment, depending on the workload the memory reclamation is put on the shared memory subsystem. 

Note that *main-memory* persistent and functional data structures (found in many functional languages) usually rely on the runtime-provided garbage collection to reclaim unused versions. Using *reference counting* to track unused blocks in Memoria may be seen as a form of deterministic garbage collection.

### Stacking on Top of Persistent Tree ###

Stacking non-persistent dynamic data structures (No-CoW) on top of persistent tree is straightforward. Memoria transforms high-level data containers to a key-value mapping in a form of BlockID->Data. This key-value mapping is then served through persistent tree, where each version is a **snapshot** or immutable point-in-time view to the set of low-level containers.

{{< figure src="cow-allocator.svg" >}}

Note that Momoria currently does not have in-house storage engines providing CoW semantics at the storage level.

### Snapshot History Graph ###

Versions in persistent tree need not to be ordered linearly. We can *branch* new version from any other *committed* version, effectively forming a tree- or graph-like structure:

{{< figure src="cow-history-tree.svg" >}}

Here, in Memoria, any path from root snapshot to a leaf (Head) is called a **branch**. A data structure is **fully persistent** (or, simply, persistent) if we can branch form any version. If we can merge versions in some way, the data structure is **confluently persistent**. Memoria doesn't use confluently persistent trees under the hood. Instead, data blocks can be copied or imported from one snapshot to another, effectively providing the same function to applications. 

If a writer may branch only from the single head, or, there is only one linear branch in the history graph, such data structure is called **partially persistent**. This apparent limitation may be useful in certain scenarios (see below).

Note that at the level of containers merging operation is not defined in a general case. It might be obvious how to merge, say, relational tables because they are *unordered sets* of rows in the context of OLTP transactions. And this type of merge can be fully automated. But it's not clear how to merge ordered data structures like arbitrary text documents or vectors. 

Because of that, Memoria does not provide complete *merge* operation at the level of snapshots. But it provides special facilities to define and check for write-write ad read-write conflicts (conflict materialization). Living the final decision and merge responsibility to applications.

Fully automated confluent persistence requires using CRDT-like schemes on top of containers.

### Single Writer Mutiple Readers ###

So-called *Single Writer Multiple Readers*, or SWMR for short, transactions can be build on the top of Memoria's snapshots pretty easy. In the SWMR scheme there is only one writer at a time but can be multiple readers accessing already committed data. To support SWMR transactions we need a lock around snapshot history branch's *head* to linearize all concurrent access to it:

{{< figure src="transactions-swmr.svg" >}}

Despite being non-scalable in theory, because of the lock, SWMR scheme for transactions may show pretty high practical performance in certain cases:

1. When write transactions are short: point-like queries + point-like updates like for moving money from one account to another. 
2. When writes does not depend on reads like in streaming: firehose is a writer ingesting events into the store, readers perform long-running analytical queries on "most recent" snapshots with point-in-time semantics.

There are two main reasons for SWMR high performance:

1. History lock is taken only for the short period of time, just to protect snapshot history form concurrent modifications. 
2. Otherwise, no additional locks are involved, except, possible, implicit locks in IO and caching subsystems of computers. 

For long-running readers the cost of these locks is amortized, so they can access dynamic shared data with efficiency of local immutable files.

### Multiple Writer Multiple Readers ###

If read+write transactions may be long, but not interfere much with each other, we can execute them in separate branches. And, *if* after completion, there are no read/write and write/write conflicts, just merge them into a single snapshot (new head). Conflicting transactions may be just rolled back automatically:

{{< figure src="transactions-mwmr.svg" >}}

This is actually how MVCC-based relational DBMS work under the hood. What we need for MWMR transactions is to somehow describe conflicting set and define corresponding merge operations for conflicting containers.

SWMR transactions are lightweight, performant and can be implemented on top of a linear snapshot history graph without much effort. But they must be short for keeping transaction latencies low. MWMR scheme can run multiple write transactions concurrently, but imply that *snapshot merging* is fast and memory-efficient. Moreover, conflict materialization also consumes space and time, and neither of these are necessary for SWMR transactions. 

### Streaming + Batching ###

As it has been said, SWMR scheme is streaming-friendly, allowing continuous stream of incoming updates and point-in-time view semantics for readers at the same time. The latter is implemented with minimum of local and distributed locks and allows read-only data access that is almost as efficient as to immutable local files.

Once snapshot is committed, it will never be changed again. So, we can run iterative algorithms on the data without expecting that this data may change between iterations. At the same time, updates can be a accumulated in upcoming snapshots. And once readers are done with current ones, they can switch to the most recent snapshot, picking up the latest changes. So, updates can be ingested incrementally and processed as soon as they arrive and ready.

## Core Storage Engines

### In-Memory Store

[IMemoryStore](https://github.com/victor-smirnov/memoria/blob/master/stores-api/include/memoria/api/store/memory_store_api.hpp) is the main, the most feature-rich *core* storage engine in Memoria, and it's the fastest option. Other storage engines *may be* limited in functionality in one or another way. It's a [confluently-persistent](https://en.wikipedia.org/wiki/Persistent_data_structure) store with CoW implemented at the level of containers.

Store is transactional (within a single branch) and multi-threaded, MWMR, and is best suitable for compute-intensive tasks when data is well-fit into memory of a single machine and *durability* of single snapshots is not required.

### SWMRStore

[SWMRStore](https://github.com/victor-smirnov/memoria/blob/master/stores-api/include/memoria/api/store/swmr_store_api.hpp) is an SSD-optimized disk-based storage engine with durable commits. It supports basically the same set of essential functions as the in-memory store, but there may be only one writing transaction active at a time. History (series of snapshots) and branches are supported. Writers do not interfere with Readers (no locks). The following list summarizes this store's features:

1. Uses reference counting for memory management. Every snapshot can be deleted from history independently from others. But see remark (*) below.
2. Optimized mainly for analytics (read-intensive ops, streaming, batching) but also for good transactional performance.
3. Supports relaxed durability when only certain snapshots are marked as durable speeding up commits on consumer-grade SSDs. In case of a crash, the store will recovered up to the last durable snapshot.
4. Support multi-phase commit protocols. SWMRStore instances can participate in *distributed transactions*.
5. Although different writers are serialized, writer itself *may be* parallel, by opening multiple (sub-)snapshots and buffering writes in memory. This mode of operation is useful for heavy-weight data transformation operations and will be supported in the future.

`Single-Writer` is actually not a *fundamental* limitation. If writers are serialized, there are pretty good algorithms for managing external (block) memory. Because of that, SWMRStore does not need an underling filesystem to allocate blocks, it may work on top of raw block devices (primary mode for a high-performance setup). MWMR mode can be implemented on top of any 'read your writes' Key/Value store, the problem is that in order to support crash recovery we will have to either scan *the entire repository* to identify orphaned blocks, or accumulate them in a separate store, merge it in at commit time and provide recovery metadata. It may have sense in some cases, but currently MWMR is not a part of the core set of storage engines for block devices.

(*) Note that reference counters are not persistent and are not stored on each commit. Because of that, SWMRStore does not provide zero-time recovery. In case of crash or improper shutdown, we have to scan the store partially to rebuild counters. Fortunately, the amount of information that needs to be scanned this way is rather small (much less that 1%) and the store is readable at this time. Counters also take main memory, about 4 bytes per block.

### OLTPStore
TBC...

### OverlayStore
TBC...

### Note on High Availability and Scale-Out
TBC...
