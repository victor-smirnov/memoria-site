---
title: "Hermes"
description: ""
lead: ""
date: 2020-10-06T08:48:57+00:00
lastmod: 2020-10-06T08:48:57+00:00
draft: false
images: []
menu:
  docs:
    parent: "overview"
weight: 20
toc: true
---

> In ancient Greece mythology Hermes is a messenger between gods and humans.

## Basic information

In the context of Memoria Framework, Hermes is a solution to the "last mile" data modeling problem. Containers can be used to model lage amouts of *higly structured* data. Hermes is intended to represent relatively small amout of *unstructured or semi-structured data* and can be used together with containers. Notable feature of Hermes as a data format is that all objects and data types have canonical *textual representation*, so Hermes data can be consumed and produced by humans. Hense, the name of the data format. 

Hermes defines an arbitrarily structured *object graph* that is allocated in a continous memory segment (or a series of fixed size segments) working as an *arena*. Top-level object is a *document*. Document is a container for Hermes *objects*. Hermes objects internally use *relative pointers*, so the data is *relocatable*. Hermes objects in this form can be:
* stored in Memoria containers, 
* memory-mapped from files or between processes, 
* embedded into an executable as a form of a *resource*, 
* sent over a network or shared between host CPU and *hardware accenerators*, even if the latter have a separate *address space*.

Hermes documents can be of arbitrary memory size, the format internally has 64-bit poiters (even on 32-bit architectures). It's fine to have TB-size documents, but the format is not intended for that. Hermes documents are *garbage-collected* with a simple *copying GC* algorithm. So the larger documents are, the more time will be spent in compactifications. Ideally, Hermes documents *should* (but not required to) fit into a single storage block, that is typically 4-8KB and may be up to 1MB in Memoria. In this case, accessing Hermes objects stored in containers will be in a *zero-copy* way.

There are three serialization formats for Hermes data.

1. **Zero** serialization. Hermes data segment use relative addresses and can be extrnalized as is, as a raw memory block. All Hermes documents support fast *integrity checking* procedure to make sure that reading foreign segments is safe. This is the fasterst format but not particularily the densest one. This format is mainly for *data storage*.
2. **Text** serialization. Human-readable, safe, and the slowest (but still fast in raw numbers). 
3. **Binary** serialization. The densest option, but faster than the textual one. Safe. Best for networking when human-readability is not a requirement.

Hermes documents are *mutable* and support run-time immutability enforcement ("make it immutable"). Immutable documents may be safely shared between threads with minor restrictions.

## Hermes Object

Hermes memory is *tagged*. A tag is a type label and has various length, from 1 to 32 bytes. Most commonly used types, like `Int` have tag length of 1 byte. Rarely used types may have tags up 32 bytes, in that case it's most likely a SHA256 *hash code* of the *type's declaration*. Hermes assumes that type hash collisions will be an extremely rare event.

Memory for tag is allocated address-wise *before* the object. Objects may have gaps between them in memory caused by alignment requirements. In that case, if a tag fits into this gap, it's allocated there. With high probability, short tags (for commonly used objects) do not take extra memory.

From the API's perspecitve, Hermes objects consist from two part. The first part is a private C++ object with verbose API conforming to certain rules. The second part is a public `View` for this object, this is much like `std::string_view` for string-like data. Mutable Hermes object receive a reference to the corresponding Document's arena allocator to allocate new data in. View object encapsulates all these complexities.

Views are *owning* in Hermes. Object's view holds a reference to the corresponding Document (for memory allocation) with a fast, *non-atomic*, reference counter. Hermes Views are nearly zero-cost, but not thread-safe.

Documents as containers have additional levels of reference counting indirection, including atomic counters. Sharing document's *data* between threads may be permitted in some cases.

### 56-bit types 

1-byte tag size has special consequences in Hermes: we are using 56-bit integers and identifiers to be able to fit the entire value into a 64-bit memory slot of a pointer. That, in many cases, saves memory.

## Datatypes

Hermes has explicit notion of a type, and Hermes types are pretty close in semantics to C++ classes, they may be *parametric* in two ways. The first way is common with C++: `MyType<Parameter>` creates new instance of `MyType` parametrized by `Parameter`. The second type of parametrization is trickier. Hermes type may have an associated *state* that is considered as a shared state for all Hermes objects of this type. Type `Decimal(10,2)` have two *type constructor* `(10, 2)` parameters: precision 10 and scale 2. Type constructor in Hermes does not create a new type, so there is no way to *statically* specialize some code for `Decimal(10, 2)`. Object instances of type `Decimal` will have a pointer to a memory, storing the corresponding type constructor's data.

Generic types in Hermes are monomorphic.

To distiguish between C++ types and Hermes types that may have type constructors, the letter are called *Datatypes*.

Datatypes are first-class objects in Hermes and the rest of Memoria. So, we may have collections of datatypes, we cap process them as data and so on. We can also parse RTTI declaration of some C++ types as *normalized* datatypes and compute thier corresponding 32-byte *type hash*. (Note that this feature is currently may be compiler-specific).

## Document

In Hermes 'document' has two meanings. 

First, `Document` is a container for Hermes data. Second, 'document' is a specific set of predefined collections organaizing objects into a tree-like structure, similar to Json. Below there is a short walk-through Hermes document text-serialization features giving us json-like experience.

### Null object 
This is the simplest document that has no object.
```json
null
```

### Generic object array
Generic array of Objects containing intgers:
```json
[1, 2, 3, 4]
```

The same generic array of Objects containing numbers of different types (integer, unsigned integer, short and float):
```json
[1, 2u, 3s, 4.567f]
```

### Typed integer array
The same array but of datatype `Array<Int>` using optimized memory layout:
```json
@Array<Int> = [1, 2, 3, 4]
```

Important note that notation above does not mean that collection `[]` will have the type specified before it. It means that we create a typed collection *from* a generic one. Parser may otimize this process by not creating the latter and supplying the data directly to the former. 

The point is that there may be may ways to create a Hermes object at parse time from hermes data. For example, notation like `"string value"@SomeDataType` is a syntactic shortcat meaning that `"string value"` will be *converted* to Hermes object of datatype `SomeDataType` at the *parse time*:

```json
"19345068945625345634563564094564563458.609"@Decimal(50,3)
```

Technically, string representation of Hermes data is not a static format, like Json, but a *domain specific language*. The purpose of this DSL is to make crafting complex Hermes data in a text form easier for humans.

### Generic map between strings and objects
Generic map between strings and objects:
```json
{
  "key1": 1, 
  "key2": [1, 2, true],
  "key3": @Array<Int> = [5,6,7,8]
}
```

### TinyObjectMap

There is a special memory-efficient variant of typed map, mapping from small integer in range [0, 51] to an Object. This map is very memory efficinet, using only 16 bytes overhead per collection. It's also very fast for reading, because it uses just one `PopCnt()` instruction (usually 1 cycle on modern CPUs) to find the slot in the hash array, given the key. Values up to 56 bits (small strings, short integers, floats) may be embedded into the hash array. Hash array has no empty slots.

Given its runtime versatility, this type of a map is used extensively to represent C-like *dynamic* structuers in Hermes, without creating a corresponding C++ objects. DSLs over Hermes may combine this type of map with *code*, resulting in dynamically typed *objects*.

## Semantic Graph

## HermesPath

## Templating engine

## Schema

## HRPC

## Interoperability with other languages

## Sources

TBC...