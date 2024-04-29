---
title: "HRPC: Hermes RPC Protocol"
description: ""
lead: ""
draft: false
images: []
menu:
  docs:
    parent: "overview"
weight: 25
toc: true
---

## Basic Information

[Hermes](/docs/overview/hermes) RPC (HRPC) is a *design space* of messaging protocols optimized for direct hardware implementation. HRPC is a pretty low-level protocol semantically adding just a little-bit over common networking models like UCP and TCP (point-to-point messaging, broadcasting streaming, etc). 

Semantically, HRPC is very similar to [gRPC](https://grpc.io) and [Cap'n-proto](https://capnproto.org/), so related skills and mental models, as well as programming patterns and paradigms are immediately applicable here. The purpose is also the same -- to simplify intreroperation between functional units in large systems. But there are three substantial differences:

1. Hermes (the main message format of HRPC) is versatile but expressive enough to be the working memory data representation. It's possible to build an entire [data-type rich programming language](/docs/overview/vm) on top of it.
1. Unlike other message formats, Hermes is *storage-oriented*, so *persisting* messages is much easier, that may enable additional use-cases.
1. HRPC is hardware-oriented, so direct hardware implementation of essential functions *is encouraged*, enabling cheap, simple but powerful middleware.

HRPC is generally *transport-agnostic*. If underling transport does not provide message ordering, HRPC stack *may* implement it. But specific HRPC implementations *may rely* on transport's provided guarantees.

HRPC has no fixed "wire format", it's a *design-space*, not a specific protocol. A minimalist point-to-point HRPC communication link may connect DDR memory controller with DDR DRAM chips using command-driven pattern, with minimal overhead comparing to a fully-native implementation. But HRPC *middleware* can also bridge and route this memory traffic via an HRPC-enabled low-latency networking to a remote disaggregated memory.

HRPC supports *streaming*, so, technically, it's a Remote *Coroutine* Call protocol, but, historically, we use abbreviation _RPC_ for that. Multithreading, lightweight (green) threading or a CPS/async [concurrency-supporting environment](https://github.com/victor-smirnov/green-fibers/wiki/Dialectics-of-fibers-and-coroutines-in-Cxx-and-successor-languages) is recommended (but not required) for implementing HRPC clients and services.

HRPC communication may be both stateful (session-aware) and stateless. It also may have feature profiles, so certain functionality (like sessions) is not required to be supported by all implementations.

The following is incomplete list of HRPC usecases. Note that all those cases support streaming in addition to classical RPC:

* Calling code in another CPU thread. Using zero-serialized Hermes/HRPC in multi-threaded applications.
* Calling code in another process (IPC). Immutable Hermes documents are position-independent, so can be shared between different address spaces.
* Calling OS kernel functions from code using [io-uring](https://en.wikipedia.org/wiki/Io_uring)-like transport. Much more convenient that syscalls in case of streaming.
* Calling device functions from code. [Computational storage](/docs/applications/storage) (database-in-a-drive) made much easier.
* Calling kernels in accelerators from code. [Processing-in/near-memory](https://semiconductor.samsung.com/news-events/tech-blog/hbm-pim-cutting-edge-memory-technology-to-accelerate-next-generation-ai/) made easier. 
* Calling CPU code from kernels in accelerators (including 'callbacks' or dynamic endpoints).
* Calling code on another machine (traditional RPC) via network middleware.
* Calling device from code or code from device on another machine.
* Calling code from device on another machine.
* Calling OS kernel functions from local/remote accelerators, remote devices, etc... OS can be run on a dedicated CPU core, leaving all other cores to applications.
* ...

## Current Implementation

HRPC is currently work-in-process, it has session-aware bidirectional communication implemented over TCP. The latter provides total message ordering that is overkill for HRPC and leads to higher latencies in case of packets loss. 

1. Can work over any messaging transport. Default (and, currently, the only) implementation is using TCP. Can work on top of QUIC and HTTP/2/3, SCTP. Can work over many *message queues*.
1. Is a session-based protocol. Session is initialized by client. For TCP transport, there is one HRPC Session per TCP connection.
1. Both Client and Server may publish *service endpoints* and call them bidirectionally via Session object. 
1. Endpoints are identified with 256-bit UIDs.
1. Protocol implementation is versatile and, together with ability to share immutable Hermes documents between threads in a zero-copy way, can be used for *safe structured inter-thread messaging*. 

See [headers](https://github.com/victor-smirnov/memoria/blob/master/runtimes/include/memoria/hrpc/hrpc.hpp) for basic API definitions. See [HRPC tests](https://github.com/victor-smirnov/memoria/tree/master/tests/hrpc) for the feature preview.

The main practical difference between HRPC and gRPC is that the former does not require a lot of code-generation for application-level data structures. Basic types are supported by Hermes itself and [structured objects](/docs/overview/hermes#tinyobjectmap) can be wrapped into flyweight C++ [handlers](https://github.com/victor-smirnov/memoria/blob/master/runtimes/include/memoria/hrpc/schema.hpp) that will be optimized away at compile-time. HRPC does not require separate in-memory representation of messages.

