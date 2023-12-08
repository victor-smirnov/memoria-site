---
title: "DSL Engine"
description: ""
lead: ""
draft: false
images: []
menu:
  docs:
    parent: "overview"
weight: 60
toc: true
---

## Introduction

Historically, Memoria was meant to be a *storage engine*, leaving data processing aspects to applications. After trying Memoria in various projects involving C++ and Java it was clear that it's not a good idea. To be used at its full potential, Memoria requires its own runtime environment. Bridging different runtimes like C++ with fibers and JVM is a close to impossible task, if efficiency is a goal. After experiments with PrestoDB on top of Memoria and development of String Data Notation format, recently superseded by [Hermes](/docs/overview/hermes/), it became clear that Memoria needs its own integrated *query execution engine*. And the technical challenge is that today's and perspective *query languages* have evolved from non-Turing-complete DSLs into fully featured *programming languages*, even with capabilities of [programming in the large](https://en.wikipedia.org/wiki/Programming_in_the_large_and_programming_in_the_small). So designing and building a kind of an SQL query execution engine (a pretty complex beast by itself, actually) would not be sufficient, given the project's long-term goals. Instead, we need a generic virtual machine, heavily tuned for various query languages.

## Related: C++ with homoiconic compile-time metaprogramming

The early experiment on this ground was an [attempt](https://github.com/victor-smirnov/jenny) to use C++ as a DSL host language by adding of a full C++ subset as a compile-time metaprogramming language into the Clang compiler. Clang is a modular and well-engineered compiler, it even can run full set of C++ in a JIT mode. Internally Clang is more like a data platform with databases and clear separation to *storage* and *compute*. Currently this experiment is on pause, we need to wait and see where evolution of C++ will go. There is substantial interest around C++ [successor languages](https://thenewstack.io/googles-carbon-among-other-potential-c-successors/) like Carbon and Cpp2/Cppfront. In addition, there is an ongoing tectonic shift in conventional commercial and opensource programming right now, induced by LLM-powered *automatic programming*. Old rationales (1960-70th) do not apply anymore, programming languages of tomorrow may look completely different in terms of focus and leading paradigms.

## What

In/for Memoria we do not try to create a new general purpose *system* language *competing* with C++ or any its successor languages. Instead, we need a highly compatible *companion* language for domains where [descriptional complexity](/docs/overview/theory/) is very high, so "*code is a new data*". 

Here is how Memoria is going to fit this niche. There are five key features:

1. Memoria defines bytecode-like extensible intermediate-level *M-code* and [Hermes](/docs/overview/hermes/)-backed *Code Model*. M-code is low-level enough to be long-term stable and suitable for *tooling* (static analyzers, etc), but high-level enough to make writing DSLs simpler.
2. Native [Hermes](/docs/overview/hermes/) integration. Hermes datatypes, objects and containers are distinct but first-class elements of the language.
3. Rich embedded metadata. Elements in the *code model* may have arbitrary Hermes metadata (annotations) associated with them.
4. Homoiconic compile-time metaprogramming and supporting *metaprogramming platform*, integrated with runtime environment and the interpreter.
5. RETE-based rule engine for advanced pattern matching.

### M-code and Code Model

Memoria does not introduce any new high-level programming language, like Java or Python. Instead, it defines intermediate-level *DSL Host* language M-code. M-code may have two forms:

1. Textual, where it looks somewhat like assembly code: classes, functions, rules, *code blocks* and statements, metadata (something like MLIR code looks like).
2. Structured representation: all of these but in a dense memory-optimized Hermes documents.

Like it's in high-level languages, there is no explicit access to the stack (no push/pop statements), but there no any *syntactic sugar*: just typed variable declarations, simple control flow (branches/cycles/exceptions) and function calls.

M-code may use pretty large subset of normalized C++ types, but internally represents them as 192-256-bit hash codes, taken from a normalized type declaration. Collisions at this level are treated as insignificantly rare. Fixed-size type name encoding simplifies and speeds up the code model significantly.

This design of M-code pursues two main goals:

1. M-code allows efficient *lowering* to C++ (or any other suitable underling language like Rust, Carbon, Mojo or MLIR/LLVM), by using mostly local transformations: each statement of M-code becomes a statement of an underling language. Depending on the underling language, M-code program may have slightly different semantics. Perfect cross-language compatibility is *not* a goal.
2. Efficient (high-performance) but lightweight *embeddable* code interpreter, suitable for evaluating DSL expressions in a C++ code.

So, M-code can be either:
1. Interpreted,
2. AOT-transpiled to C++ (maybe by building the AST directly, without parsing texts),
3. JIT-compiled to MLIR/LLVM. Probably the best way to run database queries.

1 and 2 is currently the priority. 

M-code may call native C++ code, but the latter needs to be described in a *code registry*: function type, signature, physical address, other metadata. All this information (bindings) can be automatically generated using [MBT](/docs/overview/mbt/). Native code can also call M-code, but the syntax *may be* verbose (even for the AOT-compiled M-code).

M-code functions, classes, rules (see below), metadata, embedded resources are organized into modules, modules -- into assemblies. Assemblies can be linked with executables in a read-only memory segments, assemblies are just Hermes documents. Assemblies can also be AOT-compiled into native code (libraries). This native code will run at the full speed of other Memoria code, and it will be transparently invokable from interpreted M-code.

M-code is meant to be memory/thread-safe and *UB-free* by default, but may support full unrestricted memory access in a special profile. The basic idea is simple: M-code is a *companion* (domain-specific) language to C++, so let's just move all high-performance but 'unsafe' programming to C++. At the level of M-code safety is achieved by restricting the language and using run-time checks. There is no expectations that M-code, AOT-compiled to C++, will be faster than equivalent C++ code.

### Native Hermes integration

### Rich embedded metadata

### Compile-time metaprogramming

### Rule Engine

TBC...
