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

### How it started...

Memoria started back in 2007 out of a need of having a memory-efficient multi-dimensional spatial tree for function approximation, lake [this one](/docs/data-zoo/associative-memory-2/). Contrary to traditional approaches for function approximation, like neural networks, spatial trees have much smaller computational complexity (logarithmic on average) for inference and allow computing partial and inverse functions out of the same set of parameters. Advanced data structures to the rescue, [LOUDS tree](/docs/data-zoo/louds-tree/) has 2 bits per tree node of space complexity + some small overhead. What is also important, is that LOUDS trees can be *dynamic*, allowing point-like updates, so that tree-based function approximation method can support precise in-place tuning.

### Advanced data structures

LOUDS tree internally is a [*searchable* bit vector](/docs/data-zoo/searchable-seq/) supporting two additional operations -- `rank()` and `select()` by using two additional arrays. And all of this can be implemented as a dynamic array (with logarithmic complexity of updates). The problem is that besides those two search operations and traditional point-like query and update operations, we also need dozens of service operations like *batch updates*. Implementation complexity is already skyrocketing. But besides that we also need efficient concurrent multi-threaded access, external memory, transactions and versioning. In real life, implementational complexity of even apparently simple data structure, like a bitmap, may be 100-1000 times larger that one may expect by reading its description in a textbook. What is the worst thing, you may find that you need may (dozens of) different data structures...

### Should I tried a database?

An obvious idea is to try using a database as a host... It's not that simple. Databases can be transactional, analytical or hybrid, and most of the time they are heavily optimized for *one* type of data: tabular, graph or document. Unix way: do *one* thing but do it well. Multi-model databases exist, but they are not that multi-model one may expect. Instead of one, they are doing tree things (tables, graphs and documents), and these are probably not what you what to *reuse* to speedup your development. Anyway one should definitely try this route before start even thinking about writing their own database engine. There were way less options back in 2000th when Memoria was started than now.

### Or, maybe, create a new one?

If you still want to start building your own database, prepare to suffer. Neither OS, nor programming languages are not for that. C is great, but only until you need a generic collection library. And, trust me, you will need a lot of them. Java is glorious, but prepare for low-level programming over raw memory buffers with memory leaks and undefined behaviour, or GC will be killing you every day. C++ is excellent but you will be controlling UB manually all the way down. Golang is good but its monomorphic generics story has only started recently. The same is true for Rust. High performance IO story is fully ruled by networking people who [killed fibers](https://github.com/victor-smirnov/green-fibers/wiki/Dialectics-of-fibers-and-coroutines-in-Cxx-and-successor-languages). Memory mapping hates your high-performance NVMe SSD even on reading. There is no way to *reliably* commit a transaction. Database engines are just *trying* their best in this respect. And this is, more or less, guaranteed only for certain combination of storage device, OS and drivers. And I haven't yet mentioned distributed computing. There is no reliable way to send a packet between computers. There is basically no part of your computer you can trust and rely on. And every failure may be fatal for your *state*. You want to sleep well at nights, right?

### What Memoria is...

Memoria Framework is aiming to make our life in this world easier but you may be benefited too, depending on your needs and how well they fit into the project's model.

Memoria has the following components:

1. [**Hermes**](/docs/overview/hermes) - arbitrarily structured object graphs allocated in relocatable contiguous memory segments, suitable for memory mapping and inter-process communication, with focus on data storage purposes. Hermes objects and containers are string-externalizable and may look like "Json with types". GRPC-style services and related tooling (IDL) is provided.
2. Extensible and customizable set of [**Data Containers**](/docs/overview/containers), internally based on B+Trees crafted from reusable building blocks by the metaprogramming framework. The spectrum of possible containers is from plain dynamic arrays and sets, via row-wise/column-wise tables, multitude of graphs, to compressed spatial indexes and beyond. Everything that maps well to B+Trees can be a first-class citizen of data containers framework. Containers and Hermes data objects are deeply integrated with each other.
3. Pluggable [**Storage Engines**](/docs/overview/storage) based on Copy-on-Write principles. Storage is completely separated from containers via simple but efficient contracts. Out of the box, OLTP-optimized and HTAP-optimized storage, as well as In-Memory storage options, are provided, supporting advanced features like serializable transactions and Git-like branching.
4. [**DSL execution engine**](/docs/overview/vm). Lightweight embeddable VM with Hermes-backed code model (classes, byte-code, resources) natively supporting Hermes data types. Direct Interpreter and AOT compilation to optimized C++.
5. [**Runtime Environments**](/docs/overview/runtime). Single thread per CPU core, non-migrating fibers, high-performance IO on top of io-uring and hardware accelerators.
6. [**Development Automation**](/docs/overview/mbt) tools. Clang-based build tools to extract metadata directly from C++ sources and generate boilerplate code.

The purpose of the project is to integrate all aspects and components described above into a single vertical framework, starting from *bare silicon* up to networking and human-level interfaces. The framework may eventually grow up into a fully-featured *metaprogramming platform*.

### What Memoria is not...

First of all, Memoria is not a database engine. It may contain one as a part of the Framework, but its scope will be limited (like etcd for k8s). Large-scale *distributed* storage is intentionally outside of the scope of the project.



