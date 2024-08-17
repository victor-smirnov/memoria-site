---
title: "Memoria Acceleration Architecture (MAA)"
description: ""
lead: ""
draft: false
images: []
menu:
  docs:
    parent: "overview"
weight: 65
toc: true
---

> Data dominates. If you've chosen the right data structures and organized things well, the algorithms will almost always be self-evident. Data structures, not algorithms, are central to programming.

-- Rob Pike in [“Notes on Programming in C”](http://www.lysator.liu.se/c/pikestyle.html), 1989.

## Basic Information

Processing can be compute-intensive, IO-intensive or combined/hybrid. Processing is compute intensive if each element of data is processed many times. Examples: sorting and matrix multiplication. Otherwise it's IO-intensive. Example: hashtable with random access. Hybrid processing may contain both compute-intensive and IO-intensive *stages*, but they will be clearly separable. Like, in evaluating SQL query, JOIN is IO-intensive and SORT is compute-intensive. Physically, the more processing is compute-intensive, the less it's IO-intensive. Just because while we are processing a data element intensively, we can't do IO.

_(Note that common definitions of compute-/io-/memory-bound problems are applicable here but not exactly identical to compute-/io-intensity.)_

Compute/IO-intensity is not always an intrinsic property of an algorithm, but rather a combination of algorithm and *memory architecture*. Memory architecture is *usually* a combination of memory buffers with different size and speed. Usually, *large* memory can't be *fast*. Although with current technology there may be many *partial* exceptions from this rule. From the practical perspective, by IO we mean *off-die traffic*, because it's usually 100-1000 times slower than intra-die traffic. Sometimes, on-dies memory architecture may be pretty complex, containing rather slow memory but relatively large that even *may be* considered an IO under certain conditions. But here we are not counting these cases, just for simplicity.

Each algorithm is characterized by some access pattern that can be *predictable* (e.g., linear or regular) or *random*, or mixed. In general, we can find regularities in memory access patters, either *statically* or *dynamically*, and utilize them to optimize data structures placement in memory logically and physically, we can keep most memory access intra-die, minimizing extra-die IO. 

1. Automatic strategy use memory access caching and prefetching.
1. Manual strategy use various memory buffers as a fast SRAM and move data between them manually.
1. Hybrid strategy implies using both caches and scratchpad memory, as well as explicit memory prefetching. 

Caching and prefetching is, by far, the most popular way to reduce memory access latency. It works pretty well in many practically important cases. But caching has its drawbacks too:

* Caching isn't free. *Cache miss* costs dozens of cycles and *cache hit* isn't free either (trying address lookup table). Raw scratchpad SRAM may be much faster in the *best case*.
* Caching doesn't co-exist well with Virtual Memory because it needs to take address translation into account. Switching contexts invalidate caches (either cache or address translation), that may degrade performance several times.
* Caching of mutable data isn't scalable with the number of cores because of cache-coherency traffic.
* To maximize performance under rather irregular memory access latency we need sophisticated OoOE cores, which are large, hot and expensive to engineer.

While raw DDR5 DRAM access latency is around 25-40 ns, the full system latency, or the time needed to move data through the memory hierarchy, is around 75 ns, that is more than 2 times higher. These numbers don't take into account virtual memory effects like TLB misses, which may again push the system latency up several times.

Inter-core communication is mostly done via caches and coherency traffic. One way latencies may be from 5 ns for two *virtual* cores (SMT) to [hundreds os nanoseconds](https://chipsandcheese.com/2023/11/07/core-to-core-latency-data-on-large-systems) in case of multiple sockets. Average latency is pretty high -- around dozens of nanoseconds. What is the worst, it that those numbers may be significantly higher when all cores start talking to each other. Performance may be minuscule if underling Network-on-Chip (NoC) can't handle this coherency traffic gracefully. Even garbage collection via atomic reference counting may be a pathological case for these type of memory architectures.

Existing Linux-compatible OoOE multicore CPUs, despite currently being the best way to run latency-sensitive workloads like databases, symbolic reasoners, constraint solvers, etc, aren't really being optimized for that. Slow inter-core communication makes them not that suitable for fine-grained *dynamic parallelizm*. 

## Memoria Containers

Memoria relies heavily on metaprogramming techniques used for design-space exploration of algorithms and data structures -- so called *generic programming*. In C++ we are using template metaprogramming for that, but, despite being Turing-complete, it has serious limitations. New modern programming languages like Mojo and Zig provide the full language subset for compile-time metaprogramming. [Jenny](https://github.com/victor-smirnov/jenny), Memoria's assisting Clang-based C++ compiler (used for [DSLEngine](/docs/overview/vm) and co-design tools for code targeting RISC-V accelerators) also supports calling arbitrary functions at compile time. But currently Memoria relies only on C++ template metaprogramming for design space explorations.

Memoria's [Container](/docs/overview/containers) is the main structured data abstraction unit. Containers internally have block-based structure and represented as B+Trees, either ephemeral or [persistent](https://en.wikipedia.org/wiki/Persistent_data_structure) (multi-version). Basically, any data structure that can be (efficiently) represented as an array, can be (efficiently) represented as a container. In Memoria, we use metaprogramming to build application-specific containers out of *basic building blocks* using *specifications*.

There are five [basic building blocks](/docs/data-zoo/overview), all arrays support both fixed- and variable-length elements:

1. Unsorted array.
1. Sorted array.
1. Array-packed [prefix sums tree](/docs/data-zoo/partial-sum-tree).
1. Array-packed [searchable sequence](/docs/data-zoo/searchable-seq).
1. Array-packed [compressed symbol sequence](/docs/data-zoo/compressed-symbol-seq).

Below is a schematic representation of searching through multi-ary search tree: 

{{< figure src="tree-search.svg" >}}

Each node of such search tree takes some space in memory, and the best performance is achieved when the size of the node is in low multiple of a CPU cache line: 32-128 bytes. For prefix sum trees search operation perform additions (accumulation) with comparison with each element of the node, for other tree types the operation may be different. Instead of doing this search in CPU cache (loading data there in the process), we can completely offload them either to:

* memory controller or
* to processing cores attached directly to memory banks on DRAM dies.

Embedding logic into DRAM is hard (but possible), because the process is not optimized for that. But memory parallelism (throughput _and_ latency) will be the best in this case, together with energy efficiency. This is what is called _Processing-In-Memory_ or PIM.

The alternative is to put the logic as close to the memory *chips* as possible, probably right on the memory modules or into the CXL controller. Throughput will be lower, parallelism too, latency somewhat higher. But we can leverage existing manufacturing processes, so solution will be cheaper initially. This is what is called _Processing-Near-Memory_ or PNM.

The point is that, by and large, accelerating container require as much memory parallelism as possible and corresponding number of xPU is put as close to the physical memory as possible. Existing accelerators, optimizing for neural networks (matrix multiplication), do not optimize for that (for latency). Because matrix multiplication is _latency-insensitive_. Memoria applications need a separate class of accelerators, maximizing effective *memory parallelism*.

## Persistent Data Structures

[Persistent data structures](https://en.wikipedia.org/wiki/Persistent_data_structure) or PDS are usually implemented as trees, and Memoria follows this way. PDS are very good for parallelism, because committed versions are immutable, and immutable data can easily be shared (cached) between parallel processing units without any coordination. Nevertheless, PDS require *garbage collection*, either deterministic (via atomic reference counting) or generational. Both cases require strongly-ordered message delivery and *exactly-once-processing*. The latter is pretty hard to achieve in a massively distributed environment, but rack-scale or even DC-scale (of reasonable size) is OK. Nevertheless PDS have scalability limitations that may limit even very small systems -- if they are fast enough for them to hit the limitations.

To accelerate PDS we need acceleration for atomic counters and, probably, for some other concurrency primitives. We need to make sure that communication fabric supports robust exactly-once-delivery. The latter will require some limited form of idempotent counters, that is reducible to keeping update history for a counter for a limited amount of time.

Persistent and functional data structures are *slower* than their ephemeral counterparts on a single-threaded sequential access: O(1) access time becomes O(log N) because of trees. Functional programming languages may amortize this overhead in certain cases. But it should be taken into account that benefits of using them may start manifesting only for massively-parallel applications (10+ cores).

## High-level Architecture

Computational architecture in Memoria is inherently heterogeneous and explicitly supports three domains:

1. Generic mixed **DataFlow** and **ControlFlow** computations. A lot of _practical_ compute- and IO-intensive applications, that may be run either on CPUs or on specialized hardware, fall into this category.
1. **Integrated Circuits** for fixed (ASIC) and reconfigurable logic (FPGA, Structured ASIC). May be used for high performance _and_ low power stream/mixed signal processing part of application, giving ability to handle events with a nanosecond-scale resolution.
1. **Rule/search-based** computations in a _forward chaining_ (Complex Event Processing, Streaming) and _backward chaining_ (SQL/Datalog databases) forms.

{{< figure src="tri-arch.svg" >}}

Domains are connected with a unified hardware-accelerated RPC+streaming communication protocol, [HRPC](/docs/overview/hrpc), allowing intra- and cross-domain seamless communication. HRPC is very much like gRPC (that is used to communicate with services in distributed computing tasks), but optimized for direct hardware implementation. 

HRPC, if it's implemented in hardware, eliminates the need for a fully-featured OS Kernel, reducing it to a nano-kernel, that can be as small, as the amount of HRPC functionality, implemented in the software. Memoria _computational kernel_, a program module, running on a CPU core inside an _accelerator_, can listen to a stream, generated by reconfigurable logic, running in an FPGA (and vice versa). Or, the same kernel may call storage functions running in the smart-storage device, or in a "far" memory (near-memory compute in a CXL device):

{{< figure src="comp-arch.svg" >}}

In this type of architecture, OS' kernel functionality is split into a services running on different computable devices. Storage functions, which are usually the largest OS-provided piece of functionality, are managed directly by ['smart storage' devices](/docs/applications/storage), capable of running complex DB-like queries in streaming and batching modes.

This architecture design should not be considered as a hardware-assisted micro- or nano-kernel, but rather a **distributed system scaled down to a single machine**. Large multicore MMU-enabled CPU is no longer a _central_ PU in this architecture, but rather a PU for running legacy code and code benefited from using a MMU.

Notable feature of this architecture is that memory is no longer a single shared address space, but rather a set of buffer with different affinity with compute functions. Programming it directly will be a challenge, but it's already a mundane 9-to-5 job in the area of distributed programming.

Note that in Memoria architecture, cross-environment code portability is _not ensured_. Different accelerators may provide different default runtime environments, memory and CPU cluster topologies. It's OK if some Memoria code needs a substantial rewrite to be run in a different acceleration environment. But the framework will be trying to reduce portability costs and cross-environment code duplication by using metaprogramming and other types of development automation.

## Processing Element

Reconfigurable extensible processing unit (xPU) is the main structural element of MAA. The main point of this design is that hardware-accelerated HRPC protocol is used for all communication between core and outer environment. From the outside, a core is represented as a set of HRPC endpoints described with using generic HRPC tools (IDL, schema, etc...).  This includes:

1. All external (to the core) memory traffic, including cache transfers and DMA;
1. All Debug and Observability traffic;
1. Runtime exceptions signalling;
1. Application-level HRPC communication.

Such unification allows placing xPU at any place where HRPC network is available:

* In an accelerator's cluster;
* Inside DDR memory controller;
* On a DRAM memory module (*near* memory chips, CXL-mem, PNM-DRAM);
* Inside a DRAM chip on a separate stacked die (in-package PIM-DRAM);
* On a DRAM memory *die* (PIM-DRAM);
* Inside a network router;
* < ...your idea here... >

In all cases kernel's code running in such xPUs will be able to communicate bidirectionally with the rest of environment. 

Both HRPC (low-level) and system-level endpoints specification is meant to be an open protocol, so independent manufacturers may contribute both _specialized cores_ and _middleware_ into the Framework-supported ecosystem. Software tools will be able to adapt to a new hardware either automatically or with minimal manual efforts.

Memoria code may be pretty large and have deep function call chains, so instruction cache is essential in this design (with, unfortunately, unpredictable instruction execution latencies caused by that). Another essential part of the core is 'stack cache' -- dedicated data cache for handling thread stacks. It's necessary when internal data memory is used as a scratchpad, not a D$:

{{< figure src="xpu.svg" >}}

What this architecture is not going to have, is _cache coherency_ support (unless it's really necessary in some narrow cases). MAA relies on PDS where mutable data is private to a writer, and readers may see only immutable data. If there is some shared structured mutable data access, like atomic reference counting, it can be done explicitly via RPC-style messaging (HRPC) and hardware-accelerated services.

## Accelerator Module

The whole point of MAA is to maximize utilization of available _memory parallelism_ by moving processing closer to the data, primarily for _lowering access latency_, but also for _increasing throughput_.

Ideally, every *memory bank* in the system should have either xPU or a fixed functions associated with it. Embedding logic into a DRAM die is technically challenging, although some solutions are already [commercially available](https://arxiv.org/pdf/2105.03814). Stacking DRAM and processing dies is more expensive, but may be better fit into existing processes.

The simplest way is to put xPU and fixed functions into a DRAM memory controller (MC), making it 'smart' this way. Here the logic is operating at the memory speed and has the shortest part to. No caches, switches and clock doming crossing. But processing throughput is limited comparing to what is possible with PIM mode.

So, **optimization for memory parallelism with PNM/PIM modes** and **targeting memory access latency**, not just throughput, is what makes some computational architecture good for Memoria applications.

Other than PNM/PIM and HRPC and the use of persistent data structures, Memoria Framework does not define any specific hardware architecture. Below there is an *instance* of an accelerator for a *design space* the framework will be supporting in software and tooling.

{{< figure src="accelerator.svg" >}}

Here there are following essential components:

1. Processing elements ([xPU](#processing-element)) -- RISC-V cores with hardware support for HRPC and essential Memoria algorithms and data structures.
1. Network-on-Chip (NoC) in a form of either 2D array (simpler, best for matrix multiplication) or an N-dimensional hypercube (more complex, but best for latency in general case).
1. Main HRPC service gateway and many local HRPC routers.
1. Service endpoints for hardware-implemented functions like atomic reference counting (ARC) and other shared concurrency primitives.
1. Shared on-die SRAM, accessible by all cores. It can be distributed and has many functions -- scratchpad memory, caching, ring buffers and other *hardware assisted* data structures, etc. May be physically split into many segments and distributed on the die.
1. Smart DRAM controller with embedded PNM xPUs and/or hardwired [Memoria functions](#memoria-containers). 
1. External connectivity modules (Transceivers, PCIe, etc).

The main feature of this architecture in the context of Memoria is that it's *scalable*. There are no inherent system-wide scalability bottlenecks like whole-chip cache coherence. Of course, synchronization primitives like ARC and mutexes *theoretically* aren't scalable, but they can be _practically_ made efficient enough if implemented in the hardware directly, instead of doing it in the software over a cache-coherency protocol (that we can't even control).

Other properties of this design:

1. It can be _scaled down_ to the size and power budget of an MCU and _scaled up_ to the size of an entire wafer (and beyond). 
1. It's _composable_. Memoria applications do not rely on a shared array-structured memory. They may use fast structured [transactional](/docs/overview/storage) memory for that. At the hardware level it's just bunch of chips talking to each other via an open messaging protocol.
1. It's extensible. Extra functionality can be added into xPUs (regular and custom RISC-V instruction set extensions), hardened shared functions, HRPC middleware and others. The only requirement is the use of HRPC protocol for communication via published HW/SW interfaces.

## CPU mode

As it has been noted above, multicore MMU-enabled CPUs aren't the best runtime environment for running Memoria applications because of the overhead caused by MMU, memory hierarchy and the [OS](https://github.com/victor-smirnov/green-fibers/wiki/Dialectics-of-fibers-and-coroutines-in-Cxx-and-successor-languages). Nevertheless, it's a pretty large deployment base that will only be increasing in size in the foreseeable future. And it's currently the only way to run Memoria applications.

So, Memoria Framework is going to support it as a first-class member in its hardware platform family. Once specialized hardware becomes available, it can be incrementally included into the ecosystem.

## Matrices, Tensors and Operations on Them

Many data structures can be represented as arrays. Ordinary graphs can be represented as a square matrix and, if graph is _dense_, such representation will be optimal both in terms of memory size and in terms of memory access patterns. Many algorithms on graphs can be efficiently reduced to operations on matrices. And if we operate on matrices, we can enjoy _static_ scheduling of data/control flow and, in case of matrix multiplication for instance, short data trips of systolic processing. Matrices are rarely dense, but if it's the case, benefits may be huge. 

MAA does need support for efficient matrix operations, but this entire area is currently being passionately explored in companies creating accelerators for neural networks. Solutions are very complex and highly optimized (including software side -- _compilers_), and it seems they are pretty close to the ideal.

Memoria is focusing primarily on sparse data structures processing by relying on in/near-memory computing (PIM/PNM), that is expected to reduce data access latency. There are three possible strategies how to fuse these two different types of processing in one architecture:

1. Add systolic processors/CGRAs to xPUs. They can be implemented as HRPC accessible devices to coexist peacefully with threads. But area will be wasted if this functionality isn't used.
2. Design a separate GEMM-optimized architecture in the context of MAA either in a form of specialized xPU (preferably) or even a separate accelerator module.
3. Outsource this functionality to external projects. There is substantial interest to GEMM in both open source and proprietary domains.

In all three cases hardware implementation of HRPC endpoints and middleware becomes foundational for both internal communication within MAA and with external systems. 

The whole idea of hardware HRPC is to _generate_ corresponding IP from semantically enriched IDL, much like we are doing it currently for service-oriented architectures in the context of distributed computing. Specifying interfaces in a separate place and auto-generating software (and hardware) artifacts supporting them is an efficient way to reduce solution complexity. 

Hardware implementation of HRPC (including related tools) will be a high priority for the project.

## Implementation Strategy

Implementing MAA is a major technical and organizational challenge for the Memoria project. In order to get the whole idea faster into the prototyping stage, a configurable RV ISA-level (with Memoria-specific ISA extensions, memory- and HRPC-related machinery) software emulator of MAA is planned at first. 

The emulator needs a C/C++ compiler with MAA-specific RISC-V extensions, that is also planned based on [Jenny](https://github.com/victor-smirnov/jenny) (Memoria-specific Clang's fork).

The main purpose of emulator is to start experimenting early with porting Memoria's core algorithms and data structures to MAA. Later, this emulator may be used for software development when actual hardware is not available.

The second phase is developing reference implementations of HDL IP and other related artifacts for FPGA and ASIC:

1. Hermes core algorithms and data structures as RV ISA extension instructions.
2. HRPC core protocol, transport and routing circuitry. 
3. Configurable RISC-V xPU in some pre-existing HDL.

The goal is to create a minimal, not necessary fully optimized, but functional MAA _envoronment_ for hardware developers to experiment with the framework.

There is a hardware for that:

{{< figure src="U50.jpg" >}}

The third phase is to integrate [DSLEngine](/docs/overview/vm) and [MBT](/docs/overview/mbt), the emulator into an automated development platform. 
