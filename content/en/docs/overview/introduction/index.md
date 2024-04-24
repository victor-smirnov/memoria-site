---
title: "Introduction to Memoria"
description: ""
lead: ""
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

Memoria is ... 
* a "full-stack" data-centric framework for latency-sensitive class of computations covering data- and knowledge bases, logical reasoners, constraint solving for both exact and approximate (including Any-Time) types of inference.
* based in a complexity-minded [design philosophy](/docs/overview/philosophy), derived from theorems of Algorithmic Information Theory (AIT).
* focusing on data structures across all levels of design as a main "center of complexity".
* spanning not only classical multicore CPU-centric software-only part of the design space, but also relevant hardware design space via co-design methodologies: RISC-V accelerators and direct hardware implementation of essential algorithms, data structures and communication protocols. 

Memoria Framework has the following components:

1. [**Hermes**](/docs/overview/hermes) - arbitrarily structured object graphs allocated in relocatable contiguous memory segments, suitable for memory mapping and inter-process communication, with focus on data storage purposes. Hermes objects and containers are string-externalizable and may look like "Json with types". GRPC-style services and related tooling (IDL) is provided (HRPC).
1. Extensible and customizable set of [**Data Containers**](/docs/overview/containers), internally based on B+Trees crafted from reusable building blocks by the metaprogramming framework. The spectrum of possible containers is from plain dynamic arrays and sets, via row-wise/column-wise tables, multitude of graphs, to compressed spatial indexes and beyond. Everything that maps well to B+Trees can be a first-class citizen of data containers framework. Containers and Hermes data objects are deeply integrated with each other.
1. Pluggable [**Storage Engines**](/docs/overview/storage) based on Copy-on-Write principles. Storage is completely separated from containers via simple but efficient contracts. Out of the box, OLTP-optimized and HTAP-optimized storage, as well as In-Memory storage options, are provided, supporting advanced features like serializable transactions and Git-like branching.
1. [**DSL execution engine** (DSL Engine)](/docs/overview/vm). Lightweight embeddable VM with Hermes-backed code model (classes, byte-code, resources) natively supporting Hermes data types. Direct Interpreter and AOT compilation to optimized C++.
1. [**Acceleration architecture**](/docs/overview/accel).
1. [**Runtime Environments**](/docs/overview/runtime). Single thread per CPU core, non-migrating fibers, high-performance IO on top of io-uring and hardware accelerators.
1. [**Development Automation**](/docs/overview/mbt) tools. Clang-based build tools to extract metadata directly from C++ sources and generate boilerplate code.

These components are organized into the following logical architecture:

{{< figure src="architecture.svg" >}}

The purpose of the project is to integrate all aspects and components described above into a single vertical framework, starting from *bare silicon* up to networking and human-level interfaces. The framework may eventually grow up into a fully-featured *metaprogramming platform*.

