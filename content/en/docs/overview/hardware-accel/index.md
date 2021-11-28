---
title: "Hardware Acceleration in Memoria"
description: "Hardware Acceleration in Memoria"
date: 2020-10-06T08:49:31+00:00
lastmod: 2020-10-06T08:49:31+00:00
draft: false
images: []
menu:
  docs:
    parent: "overview"
weight: 40
toc: true
---



## Introduction

Memoria was started back in 2007 after the motivation from the book [What Every Programmer Should Know About Memory](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf) by Ulrich Drepper. The idea is simple. DRAM is very slow on the random access relative to CPU speed, and while waiting for the data from memory, a CPU can perform a lot of operations. By optimizing the data layout in memory, we can reduce waiting time, improving performance. But the price is -- greater number of instruction, performed on such memory-optimized data layout. Modern out-of-order (OoO) CPUs can perform many operations each cycle, so this strategy works very well for them. The problem is that OoO CPUs scale poorly with their silicon area. Roughly speaking, from 22nm to 7nm, number of cores per consumer CPU increased 2 times. So, cores are getting bigger and bigger each iteration, but their performance is not improving that fast. By the 2020 we had 8-16 hot and large OoO cores in consumer CPUs and up to 64 cores in the server ones. But it's obvious that all this silicon budgets can be spent in a better way: to application-specific hardware accelerators. Going from 7nm to 3nm will make this situation even more obvious. Given the importance of structured data processing, it's reasonable to suppose that there is a room for Momoria-specific silicon in this budget. 

## Candidates for Acceleration

Memoria is essentially a storage technology, so it's mostly an IO-bound task. So, it's not that much to accelerate there. Nevertheless, some aspects may certainly benefit form direct hardware implementation.

1. Symbol sequences. `Select()`, `Rank()` and other operations for alphabet sizes larger than 2. Some CPUs have [`popcnt`](https://en.wikipedia.org/wiki/SSE4#POPCNT_and_LZCNT) operation support for binary alphabets, but no equivalents for larger alphabets. Software emulation is too slow. 
1. Hardware support for compressed symbol sequences (alphabet size >= 2). 
1. Memory compression and hardware tiered memory.
1. Hardware memory protection (tagged memory). Programs contain bugs, it's inevitable, which can corrupt data permanently. [Linear types](https://www.rust-lang.org) is not the ultimate solution because of unsafe code that can spread errors deep to the safe code. Again, compilers may also have errors. 
1. Fast memory checksuming: detecting corruption as soon as possible in case of hardware failures.
1. Fast memory encryption both for data storage in external memory and in the main one.
1. Native `DataType`s support, like variable length numbers, strings, unicode, etc.
1. Native support for some [LinkedData]() constructions and workflows.
1. Hardware data shuffle/scatter/gather engine.
1. Adopting [scratchpad memory](https://en.wikipedia.org/wiki/Scratchpad_memory) architecture instead of tall cache hierarchies or in addition to them.
1. Memoria-specific concurrency and parallelism modes support (hardware inter-core queues?).

Hardware data types support (sequences, integers, strings etc), hashing, compression and encryption, hardware memory protection may improve single-threaded performance, that is especially important for low-power devices. It's not expected that entire Memoria will be run on an [MCU](https://en.wikipedia.org/wiki/Microcontroller)-class device. Certain [data structures](https://bitbucket.org/vsmirnov/memoria/wiki/Memoization4AI) nevertheless may be accelerated via dedicated Memoria IP and used for inference in EdgeAI applications.

Being a storage technology, Memoria does not provide any data processing, leaving this area to applications. Nevertheless, to achieve high efficiency, storage layer and processing layer must be tightly coupled. To fulfill this need, Memoria project is providing an 'accelerator generator' -- design space exploration tool for data-intensive applications. In its core idea, Memoria itself is also a design space exploration tool, but for data structures and storage engines. Extending this idea into accelerator's area looks like a natural move. In this respect Memoria is pretty similar to the [Rocket Chip SoC generator](https://bar.eecs.berkeley.edu/projects/rocket_chip.html) for RISC-V-based systems.

## Memoria Acceleration Architecture

Acceleration Architecture is based on the [RISC-V](https://riscv.org) accelerators design pattern that is essentially an application-specific ISA extensions for a RISC core, backed with specific hardware. [Jenny Metaprogramming Platform](https://github.com/victor-smirnov/jenny) will be supporting those extensions at the C/C++ level.

1. RISC-V toolchain: compiler, binutils, simulator(s)
1. [Rocket Chip-based accelerator sandbox](https://github.com/victor-smirnov/memoria-accel)
    * Arty A7-100T board
    * Alveo U50 application accelerator
1. ISA-level simulator (not cycle-accurate) of a RISC-V-based [MPPA](https://en.wikipedia.org/wiki/Massively_parallel_processor_array) accelerator for compute-intensive applications on top of Memoria-provided data (SQL, ETL, Datalog, probabilistic programming, etc).
1. [FireSim](https://fires.im/) extensions for cycle-accurate simulation.
1. Verification tools.
1. Jenny-integrated metaprogramming tools.
