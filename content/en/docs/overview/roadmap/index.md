---
title: "Project Roadmap"
description: ""
lead: ""
draft: false
images: []
menu:
  docs:
    parent: "overview"
weight: 160
toc: true
---

## Status

After many years, Memoria is still in a permanent development. 

There are five main scopes:

1. Core: [Hermes](/docs/overview/hermes), [HRPC](/docs/overview/hrpc), [DSLEngine](/docs/overview/vm) core, related tools ([MBT](/docs/overview/mbt)), runtime-agnostic.
1. [DSLEngine](/docs/overview/vm) and [Runtime Environments](/docs/overview/runtime).
1. [Containers](/docs/overview/containers).
1. [Storage Engines](/docs/overview/storage).
1. [MAA](/docs/overview/accel).

Core is being designed to be separable and runtime-agnostic (configurable). Does not need MBT. May be distributed and used independently from the rest of the framework. May be needed when external applications want to communicate with Memoria apps, producing and consuming data in Memoria-native formats.

Hermes is _basically_ ready: API, memory layout, serialization/stringification etc. But formats haven't been finalized yet. 

HRPC is currently working over TPC/IP only that is sufficient for basic practical needs, DSLEngine is TODO and WIP.

DSLEngine is the main execution layer of Memoria, executing regular control-flow programs ("bytecode"), Datalog programs (that is also including SQL) and forward-chaining rule system based on variants of RETE algorithm ("streaming" and complex event processing (CEP)).

There are four Runtime Environments supported in Memoria:
1. Threads-based, that is mostly for running parts of Memoria alongside a legacy or external code.
1. Boost ASIO + Boost Fibers -- slower but potentially cross-platform and compatible with Boost libraries. 
1. Memoria's own Reactor engine. Single thread per core with messaging. Uses Boost Fibers for concurrency but it's own IO stack.
1. Seastar. Production ready but Linux only. Best fibers and fastest IO, but may not be the best fit for Memoria's use cases.

It's currently unknown which IO engine will be used in the future. Most likely, Memoria will be designed in an IO-agnostic way. But it will be tricky, given current struggles around lightweight threads in C++.

Containers are in a good shape but need much more work at the API level and integration with Hermes and DSLEngine. 

Storage engines. There is currently only one implementation of SWMRStore -- on top of memory mapping. OLTPStore is WIP and will be using it's own caching layer. Computational storage is TODO and slightly WIP (in the context of other subsystems).

MAA is currently at a purely conceptual stage. TODO and WIP.

## Roadmap

Hermes, HRPC and DSLEngine are the highest priority now. 

Next, deep integration of Hermes with Containers. Hermes is meant to be the main freely-structured _static_ data representation. Containers are for large and highly-structured _dynamic_ and _versioned_ datasets, including _decentralization_.

MAA is a major challenge so the plans aren't specific right now (2024). RISC-V ISA-level simulator of MAA is TODO but not yet WIP. 

HDL for Hermes and HRPC is TODO but not yet WIP. Priority is medium.

TBC...
