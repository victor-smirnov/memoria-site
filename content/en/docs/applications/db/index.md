---
title: "Converged Databases"
description: ""
lead: ""
draft: false
menu: 
  docs:
    parent: "apps"
weight: 2030
toc: true
---

Database = _storage_ + _type system_. 

The problem is that storage and type system must be co-designed. In regular programming we rely on the composability principle for build our solutions out of reusable building blocks that can be designed independently. Databases are very different: data structures (containers in Memoria), in general, are not composable under concurrent updates. We need transactions. Implementing transactions is tricky, we probably need to do it on a per-container basis (one way for tables, another way for indices, blobs, graphs, etc...)

Polyglot persistence approach, although appealing in the light of the Unix Way, isn't the answer. Tying many _different_ storage and query engines together will be a development and maintenance nightmare: linking parts of related entities in different storage engines, orchestrating transactions, handling data movements, backup-restore, software updates and upgrades, etc...

**Converged database** (here, CDB) supports all our needed data container types, indices, advanced data structures, data formats and languages and data processing paradigms (batching, streaming, top-down, bottom-up, etc...) in one system. Polyglot persistence may be needed, but, _ideally_, only for interaction with existing and legacy systems. Converged databases aren't omnipotent. They may lack certain data operation modes that don't fit easily into their narrative. So specialized databases still keep their place under the Sun, they can be just much better optimized for the narrow set of functions they perform. 

Memoria itself isn't a database, it's a frameworks for building database engines (among other things) out of provided reusable building blocks. It's closer to Erlarg & Mnesia (and actually was partially influenced of this platform's system design) than to PostgreSQL, that is moving in the direction of converged databases. True CDB should not be a traditional black box database engine accessible via uni-directed protocol like JDBC, because no size fits all. Rather, it should be a framework, platform or database engine deeply embedded into programming language (or vice versa). The main goal is that database _may not limit programmers_ in what they can imagine doable. Existing databases do, and do it badly.

Well, Memoria has the following subsystems:

* [Hermes](/docs/overview/hermes): Storage-optimized messaging data format and data modeling stack (including low-level semantic graphs).
* [HRPC](/docs/overview/hrpc): high-level bi-directional RPC-like communication protocol
* [DSLEngine](vm): (work in progress) multi-paradigm program execution engine, supporting convenient control-flow code execution, data-flow programs, generalized relational algebra, integrated RETE-based forward chain rule engine and backward-chaining Datalog-based engine (unified with all other engines), potentially accelerated with [MAA](/docs/overview/accel) (once it's available, WIP)
* [Containers](/docs/overview/containers): containers for highy-structured arbitrary sized data, support copy-on-write semantics for concurrency, parallelism, versioning and transactions. 
* [Storage Engines](/docs/overview/storage): based on Copy-on-Write principles. Storage is completely separated from containers via simple but efficient contracts. Out of the box, OLTP-optimized and HTAP-optimized storage, as well as In-Memory storage options, are provided, supporting advanced features like serializable transactions and Git-like branching and versioning.

The following diagram demonstrates a reference database engine architecture consisting from these subsystems:

{{< figure src="architecture.svg" >}}

Memoria is departing from CPU-centric architectures, adopting more distributed-systems-like approach even at the scale of a single system. This approach relies heavily on the concurrent properties of persistent data structures (wait-free access on the data path). They do have some runtime overhead comparing to good-old ephemeral data structures, but when you have hundreds and thousands of cores accessing shared data concurrently, there is no that many alternatives.

Memoria is targeting problems falling into the _latency-sensitive_/_dynamic parallelism_ category (databases, reasoners, solvers, etc...), where memory access and code execution patters are hardly predictable. Currently, it's being served buy multicore OoOE CPUs with large caches and complex memory hierarchies. Instead, Memoria will be exploring in-memory computing [approach](/docs/overview/accel) with small cores are placed _closer_ to the data they process.

The following diagram demonstrates an example of a _distributed_ Processing In-Memory (PIM)/Near-Memory (PNM) system architecture, built on top of a CPU-centric one:

{{< figure src="comp-arch.svg" >}}

Finally, as a database engine framework, Memoria will be relying on [computational storage devices](/docs/applications/storage) (CSD) concept, once these devices are available (they are WIP and high priority). CSDs will significantly reduce _complexity_ and improve reliability of data storage by coupling essential Memoria algorithms with low-level aspects of NAND (and other types of media) storage.


