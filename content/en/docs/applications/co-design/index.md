---
title: "HW/SW Co-design"
description: ""
lead: ""
draft: false
menu: 
  docs:
    parent: "apps"
weight: 2050
toc: true
---

## Basic information

By hardware/software co-design we mean a system design process when, given some functionality, we have a choice to implement it in hardware or in software. Actually, there are not that much of substantial difference between those two options, the problem is that corresponding engineering practices are _very different_. Co-design framework aims to align them to each other and, if possible, unify into a single seamless _super-process_.

We have practically successful examples of existing co-design frameworks -- neural networks-oriented _opensource_ ML frameworks (PyTorch, Tensorflow and others) support using custom designed hardware to accelerate computational graphs. Independent hardware manufactures can adapt their hardware to the ways how ML framework operate, and ML frameworks _users_ may design custom hardware to accelerate their _specific_ dataflow graphs. 

Memoria follows the same paradigm: it defines formal computational environment, protocols, data structures and their implementations, domain-specific languages and compilers, but is focused on a different class of computations than existing frameworks -- on _latency-sensitive_ computations and on _memory parallelism_. 

High-level computation model in Memoria is [event-driven computations](https://en.wikipedia.org/wiki/Event-driven_programming). There are three corresponding _hardware domains_ in the focus of Memoria's:

{{< figure src="/docs/overview/accel/tri-arch.svg" >}} 

* Generic mixed **DataFlow** and **ControlFlow** computations. A lot of _practical_ compute- and IO-intensive applications, that may be run either on CPUs or on specialized hardware, fall into this category.
* **Integrated Circuits** for fixed (ASIC) and reconfigurable logic (FPGA, Structured ASIC). May be used for high performance _and_ low power stream/mixed signal processing part of application, giving ability to handle events with a nanosecond-scale resolution.
* **Rule/search-based** computations in a _forward chaining_ (Complex Event Processing, Streaming, Robotics) and _backward chaining_ (SQL/Datalog databases) forms.

Functions in hardware domains are connected with a unified hardware-accelerated RPC+streaming communication protocol, [HRPC](/docs/overview/hrpc), allowing intra- and cross-domain seamless communication. HRPC is very much like gRPC (that is used to communicate with services in distributed computing tasks), but optimized for direct hardware implementation.

Generalized computational model in Memoria is _data-flow_ over decentralized _persistent data structures_. Data-flow can be either a plain low-level tokenized one or high-level complex-event-driven [RETE-based](https://en.wikipedia.org/wiki/Rete_algorithm) one.

As a _co-design framework_, Memoria defines two main design _extension points_:

1. Custom hardware functions attached via bridges to an HRPC router and visible in the system as HRPC endpoints. This is _loose coupling_ design pattern.
1. Custom ISA extensions for a [RISC-V reconfigurable processing unit](/docs/overview/accel/#processing-element). This is _tight coupling_ design pattern.

Loose coupling of functionality simplifies compositionality and _distribution_ (of compute within a memory architecture, for example). Tight coupling pattern allows _concentration of compute_ where necessary (matrix multipliers for ANN/HPC).

At the software level, Memoria is also a _software development framework_ (different one) for data intensive applications like data platforms, storage systems, networking and IoT/embedded. The whole idea is that by following Memoria-defined protocols hardware developers my gain immediate access to a pretty complex software ecosystem.

Co-design part of the framework consists from the following components:

1. [Hermes](/docs/overview/hermes) and [HRPC](/docs/overview/hrpc) libraries and tools, this is also a part of Memoria Core module. Core can be used independently from the rest of Memoria.
1. [DSLEngine](/docs/overview/vm) provides runtime-efficient system-wide _Intermediate Representation_ of the code model for each _hardware domain_. IC domain may be lowered to [Circt](https://circt.llvm.org/) (preferable way).
1. Clang-based [Jenny C++ compiler](https://github.com/victor-smirnov/jenny) and associated tools targeting CF/DF/Rule domains and custom RISC-V kernels. Note that regular Memoria applications don't need a dedicated compiler. Any compliant C++20 compiler should be OK. Clang is preferable.
1. Build and automation tools. Generic programmable event-driven build system, available both in command line and via HRPC API. HRPC may have JSON and gRPC bindings for compatibility reasons.

## Special features

As a special feature, Memoria provides data containers for [compressed searchable sequences](/docs/data-zoo/compressed-symbol-seq) with alphabets from 1 to 8 bits. These data structures are good for capturing digital waveforms for further analysis: 

1. Large alphabet is needed when signals may have more than 2 states.
1. Searchbility is needed for locating specific patterns in the signal.





