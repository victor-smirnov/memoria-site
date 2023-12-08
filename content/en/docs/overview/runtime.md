---
title: "Runtime Environments"
description: ""
lead: ""
draft: false
images: []
menu:
  docs:
    parent: "overview"
weight: 50
toc: true
---

Memoria tries to be a runtime-agnostic wherever it's possible. There are two main dependencies: memory allocation and IO. Custom memory allocation is a rather easy thing. High-performance IO is much harder because we need facilities which are [not properly supported](https://github.com/victor-smirnov/green-fibers/wiki/Dialectics-of-fibers-and-coroutines-in-Cxx-and-successor-languages) by the programming language, operation system or both. Memoria relies on fibers for concurrency and they are expected to be [supported](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p0876r13.pdf) for C++26 and is already available in various frameworks and libraries.

The goal of Runtime Environment (RE) is to provide efficient and deterministic UB-free code execution. Framework relies on the thread-per-core/message-passing model running non-migrating fibers. Resumable functions (aka 'coroutines') will be supported where appropriate, but they are viral and allocate dynamic memory for frames all the way down. Because of frame allocation, resumable functions are not as performant in C++, as it may be expected.

There are 2.5 RE 'backends' in Memoria:

1. [Seastar](https://seastar.io/) framework. Best-in-class, high performance and well-supported. But _Linux-only_. 
2. Boost Asio. Pretty high performance, perfect compatibility with various runtimes, cross platform, but no high-performance disk IO support.
3. Reactor Engine. Initially it was meant to be in-house *cross-platform* variant of Seastar with Boost Finer support. But later it lost its momentum and currently is meant to be replaced with combination of Seastar and Asio. 

Memoria is not going to rely 100% on the Seastar or Asio API. Instead, the Framework will be trying to isolate specifics of underling API, making code porting easier. 100% cross-RE compatibility is not a goal.

_This section is currently deeply incomplete, and looks like a mess, as well as a current state of runtime subsystem in Memoria. The section will be co-improving together with the runtime environment._
