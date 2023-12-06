---
title: "Containers"
description: ""
lead: ""
date: 2020-10-06T08:48:57+00:00
lastmod: 2020-10-06T08:48:57+00:00
draft: false
images: []
menu:
  docs:
    parent: "overview"
weight: 30
toc: true
---

## Basic Information

Containers in Memoria are basic units of data modelling. Idiomatically, they are very much like STL containers except that, counterintuitively, container objects do not *own* their data, they use dedicated storage API for that. So data life-cycle is tied to the *storage* object, not the *container* object. In some cases (implementations) container objects may own their storage objects, but, typically, they don't.

Note that there may be some terminological clash between Memoria containers discussed here and [Hermes](/docs/overview/hermes) containers. The latter are classical STL-like containers owning their data.

Memoria containers are block-organized. Blocks may be of arbitrary size, up to 2GB, there are no inherent limitations, but their size is typically multiple of the storage/memory block size. Like, 4K, 8K, ... The upper limit is storage-dependent. Disk-based storage engines do not allow blocks over 1MB in size for practical reasons.

[Packed Allocator](/docs/data-zoo/packed-allocator/) is a mechanism that is used in Memoria to place data (allocate objects) in memory blocks.

Blocks are organized into linked data structures using *block identifiers*. Conceptually, an identifier may be of arbitrary *fixed size* type (integer, UUID, ...), but Memoria provides dedicated type set for that. 

The most common linked data structure used for containers is a *variant* of B+Tree. This variant is mainly different from a [standard one](https://en.wikipedia.org/wiki/B%2B_tree) is that there are no *sibling links*. There are also no *parent links* in the tree, this is necessary for [persistence](https://en.wikipedia.org/wiki/Persistent_data_structure). So, in some cases B+Trees in Memoria will be less efficient than standard ones.

Lack of *sibling links* is not a big issue, because tree-walking overhead for B+Trees with large (4K+) blocks is pretty moderate. 

Lack of *parent links* is more impacting, because iterators now need to keep *full path* form root to the current node. Iterator is a stack-like data structure, not just a current block ID. A lot fo tree-updating code becomes much more complicated comparing to the variant with parent links, but this is the price we pay for having persistence, concurrency and parallelism.

## Concurrency and Parallelism

Containers in Memoria are **not** thread safe, and this is foundational design decision to make data structures simpler. All thread-safety, if any, are provided at the level of storage engines. And the main concurrency and parallelism mechanism Memoria relies on is [persistent/functional](https://en.wikipedia.org/wiki/Persistent_data_structure) data structures. This feature also comes with its design costs, limitations and overheads. But it also gives Memoria all of its batteries and superpowers.

So, B+Tree-based containers in Memoria can be of two implementation types:

1. **Copy-on-Write-based (CoW)** containers. Persistence is supported at the level of containers. This is the fastest option but at the expense of more complicated container design. All container types need to align with CoW semantics, that is well-encapsulated by the Framework.
2. **Epithemeral (non-CoW)** containers. This type of containers do not explicitly support CoW semantics themselves, so the *may* have parent and sibling links if necessary. But for the sake of code unification and reuse, they *don't*. CoW semantics may still be supported at the level of *storage engines*. For example, there is a variant of storage engine on top of the [LMDB](http://www.lmdb.tech/doc/) database that has strongly serialized CoW-based transactions. When working on top of such storage engines, containers do not need to provide their own CoW semantics. 

## Storage-agnostic

From a container's perspective, block storage is completely decoupled and can be fully software-defined.

{{< figure src="io.svg" >}}

For more details on that see the [Storage engines](/docs/overview/storage/) section.

## Definitive Example

```c++
// We will be using in-memory store
#include <memoria/api/store/memory_store_api.hpp>
// And a simple Set<> container
#include <memoria/api/set/set_api.hpp>
// Static initialization stuff
#include <memoria/memoria.hpp>

using namespace memoria;

int main(void) {
    // Create a store first. All data is in a store object.
    // We will be using an in-memory store
    auto store = create_memory_store();

    // In-memory store is a confluently-persistent CoW-enabled store,
    // so it suppoerts Git-like branching for containers.
    // We are opening new snapshot from the master branch.
    auto snapshot = store->master()->branch();

    // Now let's create a container, it will be a set of short strings.
    // First, we need to define a datatype for container:
    using DataType = Set<Varchar>;
    // See Hermes docs for more information about datatypes.

    // Now lets create a new set container in our snapshot
    auto set_ctr = create<DataType>(snapshot, DataType{});

    // .. and insert a few strings into it
    for (size_t c = 0; c < 10; c++) {
        set_ctr->upsert(std::string("Entry for ") + std::to_string(c));
    }

    // Now we are ready to iterate over inserted entries.
    set_ctr->for_each([](auto entry){
        println("{}", entry);
    });

    // After we are done with inserting data, we can commit the snapshot
    // so it will be avaliable for other threads to branch from.
    snapshot->commit();

    // And we can store the data into a file
    store->store("set-data.mma");

    // Here all the data will be destroyed in memory,
    // but will remain in the file
}

```

This example mentions using [Hermes datatypes](/docs/overview/hermes/#datatypes). Container type is defined by its *Datatype*, that is, basically a combination of C++ class and some *state*, that will be shared between all objects of this datatype. Note that all datatypes in Memoria are currently stateless, so they look exactly like C++ classes.

Working code this this example can be found [here](https://github.com/victor-smirnov/memoria/blob/master/examples/simple_set.cpp).

## Compositionality and Instantiation of Containers

STL containers are pretty lightweight and are instantiated at the site of usage. They support composition, so we can easily define composite multimap container like `std::map<int64_t, std::vector<std::string>>`. Memoria containers use b+trees aiming to support external memory and have no such luxury of composition. Moreover, arbitrary C++ objects like `std::string` *can't be* used as datatypes, because their memory management is incompatible with internal b+tree machinery. So, instead of STL's compositionality, Memoria provides complex, datatype-optimized containers. Like, STL implementation of multimap becomes `Multimap<Int64, Varchar>`. Framework provides complex containers for many practical use cases.

Memoria containers are instantiated in a library and usually comes in a pre-compiled form. The problem is that pre-instantiation of container for all combinations of datatypes is infeasible. Memoria libraries provide the most idiomatic and commonly used ones, and applications are free to instantiate their own variants, it's fully supported.

The following line may look like an instantiation of a template at the call site:

```c++
auto set_ctr = create<DataType>(snapshot, DataType{});
```

But there is no actual instantiation here. Instead, datatype's metrics (typehash code) is used to find actual instantiation in the instantiations registry.

In other for this mechanism to be really lightweight at the call-cite and work properly, containers need to have abstract *public* virtual interfaces and *private* implementations that are hidden from the application code.

Containers have pretty complex lifecicle and metadata systems, as well as integration with Hermes and datatypes, so developing a new container may be a challenge. Memoria uses in-house [Build Tool](/docs/overview/mbt/) to automate these processes.

## Metaprogramming Framework

This is one of the most complex part of the Framework, especially because C++ isn't that good with *metaprogramming in large*. There were no much better alternatives back in the days when Memoria started, there are not that many of them now. This issue is going to be addressed low-level languages that support homoiconic compile-time metaprogramming, like Zig and (as it's being promised) Mojo. Memoria's [DSL subsystem](/docs/overview/vm) is also addressing this issue too. Nevertheless, for container-level metaprogramming we currently only have what *latest* C++ standard is offering.

Memoria Containers are build using Partial Class programming pattern. Partial Class definition may be split between many files. Containers classes are built from [parts](https://github.com/victor-smirnov/memoria/tree/master/containers/include/memoria/containers/set/container) using the information provided from three places:

1. Container [prototype](https://github.com/victor-smirnov/memoria/tree/master/containers/include/memoria/prototypes). Prototype is a complex, Memoria-specific, form of a *base class*.
2. Container Datatype, like `Set<Varchar>`.
3. [Profile](https://github.com/victor-smirnov/memoria/tree/master/containers-api/include/memoria/profiles). Profiles are an elaborate system of type traits (configurations) that define various *common* parameters, like CoW/non-Cow, type of block identifier, type of snapshot identifier and so on. Dozens of them.
4. [Type Factory](https://github.com/victor-smirnov/memoria/blob/master/containers/include/memoria/prototypes/bt/bt_factory.hpp) is a metaprogramming engine building container classes out of all this stuff above.

## Source code entry points

1. [Public Containers API](https://github.com/victor-smirnov/memoria/tree/master/containers-api/include/memoria/api). 
2. [Private Containers Implementations](https://github.com/victor-smirnov/memoria/tree/master/containers/include/memoria/containers)
3. [Default Container Instantiations](https://github.com/victor-smirnov/memoria/blob/master/codegen/include/codegen_memoria.hpp) (Build Tool metadata)

