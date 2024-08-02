---
title: "Introduction to Memoria Framework"
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

Memoria is a hardware/software co-design framework for solving data-intensive problems. This scope includes:
* Transactional, analytical and hybrid [database engines](/docs/applications/db) of various kinds.
* Storage engines, including decentralized and [Computational storage](/docs/applications/storage).
* Reasoning engines for [Hybrid](/docs/applications/aiml) and Symbolic AI.
* Programming languages and compilers/tools.

HW/SW co-design means that Memoria does not imply anymore that its algorithms and data structures are run on CPU-centric architectures: single shared address space memory, multi-core architectures with coherent caches and interaction via memory and so on. While such architectures are first-class support targets for Memoria, in order to unleash its full potential we need to be able to design custom computational, memory and storage architectures, optimized for Memoria.

Memoria Framework has the following components:

1. [**Hermes**](/docs/overview/hermes) - arbitrarily structured object graphs allocated in relocatable contiguous memory segments, suitable for memory mapping and inter-process communication, with focus on data storage purposes. Hermes objects and containers are string-externalizable and may look like "Json with types". GRPC-style services and related tooling (IDL) is provided (HRPC).
1. Extensible and customizable set of [**Data Containers**](/docs/overview/containers), internally based on B+Trees crafted from reusable building blocks by the metaprogramming framework. The spectrum of possible containers is from plain dynamic arrays and sets, via row-wise/column-wise tables, multitude of graphs, to compressed spatial indexes and beyond. Everything that maps well to B+Trees can be a first-class citizen of data containers framework. Containers and Hermes data objects are deeply integrated with each other.
1. Pluggable [**Storage Engines**](/docs/overview/storage) based on Copy-on-Write principles. Storage is completely separated from containers via simple but efficient contracts. Out of the box, OLTP-optimized and HTAP-optimized storage, as well as In-Memory storage options, are provided, supporting advanced features like serializable transactions and Git-like branching.
1. [**DSL execution engine** (DSL Engine)](/docs/overview/vm). Lightweight embeddable VM with Hermes-backed code model (classes, byte-code, resources) natively supporting Hermes data types. Direct Interpreter and AOT compilation to optimized C++.
1. [**Memoria Acceleration Architecture** (MAA)](/docs/overview/accel). HW/SW co-design architecture and methodology targeting Memoria applications (see above).
1. [**Runtime Environments**](/docs/overview/runtime). Single thread per CPU core, non-migrating fibers, high-performance IO on top of io-uring and hardware accelerators.
1. [**Development Automation**](/docs/overview/mbt) tools. Clang-based build tools for extracting metadata directly from C++ sources and generating boilerplate code.

These components are organized into the following logical architecture:

{{< figure src="architecture.svg" >}}

The purpose of the project is to integrate all aspects and components described above into a single vertical framework, starting from *bare silicon* up to networking and human-level interfaces. The framework may eventually grow up into a fully-featured *metaprogramming platform*.

