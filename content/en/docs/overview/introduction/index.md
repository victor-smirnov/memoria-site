---
title: "Introduction to Memoria"
description: ""
lead: ""
date: 2020-10-06T08:48:57+00:00
lastmod: 2020-10-06T08:48:57+00:00
draft: false
images: []
menu:
  docs:
    parent: "overview"
weight: 10
toc: true
---

> Data dominates. If you've chosen the right data structures and organized things well, the algorithms will almost always be self-evident. Data structures, not algorithms, are central to programming.

-- Rob Pike in [“Notes on Programming in C”](http://www.lysator.liu.se/c/pikestyle.html), 1989.

### What Memoria is...

_Database_ = _storage_ + _type system_. This definition isn't strict, because a lot of designs can fall into this category under certain angle of sight. The distinctive property of a database is that the storage is _expected_ to be _reliable_. Reliable storage is defined by current _memory architecture_ and type system is defined by the _problem_ being solved (by algorithms and data structures). The problem is that both components are _rapidly evolving_, but the speed and consistency of this evolution is not uniform:

1. Our OS-provided IO interfaces are modelled after _tape_ storage. 
2. Modern solid-state drives have to provide block device interface to an OS that translates it to API originally intended for tape drives. Building up multiple abstraction layers on the go.
3. Databases somehow have to provide _fast_ and _reliable_ storage on top of this.
4. At the same time by using software architecture optimized for _rotating disks_ from 1980-90th.
5. At the same time hardware is heterogeneous, with hundreds of CPU cores per machine, with multiple tightly connected hardware accelerators and complex memory hierarchies, with computations offloading directly into main memory and storage devices' chips.
6. Emerging _hybrid_ AI is dictating completely new requirements for the entire HW/SW stacks. "Vector databases" are just the beginning.
7. Popular programming languages are optimized for simple _main-memory_ data structures like arrays. Anything more complex requires data and process abstractions neither these languages, not their _runtimes_ can handle efficiently.

Recent advancements in generative AI sparkled interest in GOFAI systems on the expectations that the latter can provide required level of accuracy and reliability for practical applications. Relational databases are remnants of deductive databases of 1980th that lost their "deductive" part. SQL is based on relational algebra that is based on relational calculus, that is a decidable subset of first-order logic (FOL) that can be evaluated in a polynomial time. If someone is doing a ore-or-less generic logical reasoning engine, they are doing a database, like it or not. Because relational calculus is a subset of FOL, so they end up building the same machinery needed for SQL query evaluation.

Memoria isn't a perfect and ultimate answer for those challenges. It optimizes for data variety by providing a multitude of different high- and low-level data structures ("containers") optimized into a single vertically integrated data engineering framework for SW/HW co-design of storage, compute and algorithms:

1. Memoria relies on C++ metaprogramming for data structures' _design space exploration_ ("type system", see above).
2. These data structures are used to build multiple _storage engines_ optimized for OLTP and HTAP workloads ("storage").
3. _Runtime Environment_ optimized for hardware-accelerated query execution in backward-, forward-, hybrid-chaining and streaming (continuous) modes and DSL hosting stack.

### Motivating example: advanced spatial indexes

Memoria started back in 2007 out of a need of having a memory-efficient multi-dimensional spatial tree for function approximation, like [this one](/docs/data-zoo/associative-memory-2/). Contrary to traditional approaches for function approximation, like neural networks, spatial trees have much smaller computational complexity (logarithmic on average) for inference and allow computing partial and inverse functions out of the same set of parameters. Advanced data structures to the rescue, [LOUDS tree](/docs/data-zoo/louds-tree/) has 2 bits per tree node of space complexity + some small overhead. What is also important, is that LOUDS trees can be *dynamic*, allowing point-like updates, so that tree-based function approximation method can support precise in-place tuning.

LOUDS tree internally is a [*searchable* bit vector](/docs/data-zoo/searchable-seq/) supporting two additional operations -- `rank()` and `select()` by using two additional arrays. And all of this can be implemented as a dynamic array (with logarithmic complexity of updates). The problem is that besides those two search operations and traditional point-like query and update operations, we also need dozens of service operations like *batch updates*. Implementation complexity is already skyrocketing. But besides that we also need efficient concurrent multi-threaded access, external memory, transactions and versioning. In real life, implementational complexity of even apparently simple data structure, like a bitmap, may be 100-1000 times larger that one may expect by reading its description in a textbook. What is the worst thing, you may find that you need many (dozens of) different data structures...

### The project's structure

Memoria Framework is aiming to make our life as a data/storage engineers in this world easier but you may be benefited too, depending on your needs and how well they fit into the project's model:

{{< figure src="architecture.svg" >}}

Memoria has the following components:

1. [**Hermes**](/docs/overview/hermes) - arbitrarily structured object graphs allocated in relocatable contiguous memory segments, suitable for memory mapping and inter-process communication, with focus on data storage purposes. Hermes objects and containers are string-externalizable and may look like "Json with types". GRPC-style services and related tooling (IDL) is provided (HRPC).
2. Extensible and customizable set of [**Data Containers**](/docs/overview/containers), internally based on B+Trees crafted from reusable building blocks by the metaprogramming framework. The spectrum of possible containers is from plain dynamic arrays and sets, via row-wise/column-wise tables, multitude of graphs, to compressed spatial indexes and beyond. Everything that maps well to B+Trees can be a first-class citizen of data containers framework. Containers and Hermes data objects are deeply integrated with each other.
3. Pluggable [**Storage Engines**](/docs/overview/storage) based on Copy-on-Write principles. Storage is completely separated from containers via simple but efficient contracts. Out of the box, OLTP-optimized and HTAP-optimized storage, as well as In-Memory storage options, are provided, supporting advanced features like serializable transactions and Git-like branching.
4. [**DSL execution engine** (DSL Engine)](/docs/overview/vm). Lightweight embeddable VM with Hermes-backed code model (classes, byte-code, resources) natively supporting Hermes data types. Direct Interpreter and AOT compilation to optimized C++.
5. [**Runtime Environments**](/docs/overview/runtime). Single thread per CPU core, non-migrating fibers, high-performance IO on top of io-uring and hardware accelerators.
6. [**Development Automation**](/docs/overview/mbt) tools. Clang-based build tools to extract metadata directly from C++ sources and generate boilerplate code.

The purpose of the project is to integrate all aspects and components described above into a single vertical framework, starting from *bare silicon* up to networking and human-level interfaces. The framework may eventually grow up into a fully-featured *metaprogramming platform*.


