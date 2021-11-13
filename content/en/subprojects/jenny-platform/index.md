---
title: "Jenny Metaprogramming Platform"
description: ""
lead: ""
date: 2021-10-28T02:06:54-04:00
lastmod: 2021-10-28T02:06:54-04:00
draft: false
images: []
menu: 
  subprojects:
    parent: "subprojects"
weight: 40
toc: true
---

See [Why C++](/docs/overview/whyc++) document for introduction.

## Better C++ ##

C++ TMP, though pretty powerful, has its fundamental limitations. The following is the list of the most annoying ones. They are not about the language itself, but the productivity of the language in a programming-in-large context. 

1. Template metaprograms can't do any IO, so any input data structures must be represented as lists of types (and plain arrays of literal types). There is no convenient way to apply schema to a, for instance, hierarchical data. C++ concepts to the rescue, but they just make the problem more obvious. 
1. Very limited set of data structures that can be processed with constexpr functions. Constexpr functions are interpreted under the hood and operating with raw C++ AST objects. They are slow and consume a lot of memory. 
1. Template metaprograms are almost impossible to debug. Constexpr functions are not that much better in this respect. 
1. No AST-level macro system. Embedding DSLs is severely limited relative to modern languages. 
1. Archaic build system, which the language itself has no integration with. Decent automation above C++ is a tricky thing, at least.
1. Many things would be way simpler, if C++ had modern Reflection capabilities instead of RTTI. 

By the end of 2019 Memoria hit the limits of TMP. Datatypes have been introduced that required a lot of additional metadata at compile time and run time. As well as a separate DSL for datatype description. Maintaining all this metadata using only native C++ facilities (typelist-based type descriptions) became infeasible. 

### Jenny Platform

But instead of applying partial solutions like external code-generators to C++, [the Jenny Metaprogramming Platform](https://github.com/victor-smirnov/jenny), deeply integrated with C++ compiler and the build system, had been proposed. Jenny is based on experimental [Reflection TS proposal](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1240r1.pdf) adding the ability to call ordinary C/C++ functions at the compile time val the `metacall()` operator. Unlike constexpr functions, Jenny's metafunctions can utilize full set of C++ language and run-time objects: doing I/O, spawning threads, accessing lexical context and the compiler's APIs.

### Metafunctions

Both Clang and GCC allow running compiler plugins having access to entire translation unit's AST. These plugins can be used to analyze and transform the code. But interaction between source code and a plugin is seriously limited. Jenny's metafunctions can be seen as a flexible API between the compiler and the source code it is processing. Instead of writing a compiler plugin, it's possible to implement the same functionality with metafunctions. Speaking about Clang, its plugin API is seriously limited and intended mostly for static analyzers. Jenny's metafunctions have much more flexible access into AST and, therefore, they are more powerful.

Below there is an non-exhaustive list of what Jenny's metafunctions can do:

1. Take and return all values that regular constexpr functions can work with. Wrapping regular function into `metacall()` operator makes it callable at compile-time. This way we, for instance, can use `printf()` to dump some debug info during compilation (including source code of programmatically-created types and functions).
1. Have access to the compiler-provided API for metafunctions:
    * access lexical context of the metafunction's call site;
    * parse raw strings with C++ code snippets creating AST objects that can be manipulated further. If such AST object is returned from a metafunction, it's being inserted into the proper place in the target AST the metafunction was invoked from.
1. Have access to Builder API, Project API and CompilationCache API. Compilation Units are aware about the context they are being compiled in.
1. Prepare runtime-available reflection and annotation metadata in the form of byte-array-mapped [LinkedData](https://bitbucket.org/vsmirnov/memoria/wiki/String%20Data%20Notation).
1. Access databases and external structured datasets. 
1. Spawn threads, do any allowed IO, call external libraries like theorem provers and constraint solvers. Metafunctions is a natural tool to integrate project-specific AI instruments into the development process.

### Integrated Build Tools

Besides metafunctions, Jenny is providing integrated build tool (JBT), that is using Memoria internally as a domain data model. It can be seen as "make on steroids", but otherwise it's a fully featured generic batch processing data platform with pluggable storage engines, service-oriented architecture design and RESTful API to be a [Language Server](https://langserver.org/) *backend* for Memoria/Jenny projects. Metaprograms require first-class support from the build tool to be actually usable in interactive (IDE-centric) workflows.

Unlike modern data platforms, JBT can be as versatile as the `make` tool. No any special setup is required for classical use-case scenarios.
