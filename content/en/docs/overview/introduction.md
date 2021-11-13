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

## What Memoria is

Memoria is a full-stack data engineering framework aiming at exploiting inherent structure in data at all scales, starting form bare fabric or reality and ending at high-level visualizations. See [definitions](/docs/overview/definitions) for the quick overview of philosophy and math behind Memoria.

## Target Applications

1. Hardware-accelerated Transactional Databases and Storage Engines.
1. Filesystems.
1. Analytics.
1. Artificial Intelligence and Machine Learning.
1. Software and hardware development tools.
1. And any combination of the above.

## Project structure

Main implementation language is modern C++ (14/17/20). Python bindings are also provided, mainly for ad-hoc manipulation with a data.

1. Structured data is exposed to applications via **Containers** API.
1. Containers are managed by a **Store**. It's an API that may have multiple implementations, like in-memory or on-disk.
1. Computations over data are described via low-level **Dataflow graph**-like language, that may have multiple compilation targets (CPU, Accelerators, FPGAs/ASICs).
1. The project's structure is deeply automated with dedicated **Memoria Build Tool (mbt)**, on top of Clang libraries and Python scripts.
1. Additional **tools**, like [Datascope](/docs/datascope/overview) to accommodate development process.
1. Rich set of highly customizable generic data containers and dataflow templates for various narrow fields (SQL Analytics, AI and ML, probabilistic programming, etc).
1. High-performance cross-platform (Windows, Linux, MacOSX) runtime environment with Asynchronous IO and Fibers.

## What Memoria is NOT

Memoria itself is not a DBMS, because _no size fits all_. Instead, it can be used to build a memory-focused computational infrastructure (both at the hardware and at the software levels), fitting the needs of specific applications. 

Nevertheless, [SwimmerDB](/subprojects/swimmer-db/) is a reference implementation of a Memoria-enabled scaleup-oriented (embedded, single-host or small-cluster) database engine, mostly for prototyping data structures, but also for real applications if the database fits their requirements.
  
