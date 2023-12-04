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

Containers in Memoria are basic units of data modelling. Idiomatically, they are very much like STL containers except that, counterintuitively, container objects do not *own* their data, they use dedicated storage API for that. So data life-cycle is tied to the *storage* object, not the *container* object. In some cases (implementations) container objects may own their storage objects, but, typically, they don't.

Note that there may be some terminological clash between Memoria containers discussed here and [Hermes](/docs/overview/hermes) containers. The latter are classical STL-like containers owning their data.

Memoria containers are block-organized. Blocks may be of arbitrary size, up to 2GB, there are no inherent limitations, but their size is typically multiple of the storage/memory block size. Like, 4K, 8K, ... The upper limit is storage-dependent. Disk-based storage engines do not allow blocks over 1MB in size for practical reasons.

Blocks are organized into linked data structures using *block identifiers*. Conceptually, an identifier may be of arbitrary *fixed size* type (integer, UUID, ...), but Memoria provides dedicated type set for that. 

The most common linked data structure used for containers is a *variant* of B+Tree. This variant is mainly different from a [standard one](https://en.wikipedia.org/wiki/B%2B_tree) is that there are no *sibling links*. There are also no *parent links* in the tree, this is necessary for [persistence](https://en.wikipedia.org/wiki/Persistent_data_structure). So, in some cases B+Trees in Memoria will be less efficient than standard ones.

Lack of *sibling links* is not a big issue, because tree-walking overhead for B+Trees with large (4K+) blocks is pretty moderate. 

Lack of *parent links* is more impacting, because iterators now need to keep *full path* form root to the current node. Iterator is a stack-like data structure, not just a current block ID. A lot fo tree-updating code becomes much more complicated comparing to the variant with parent links, but this is the price we pay for having persistence, concurrency and parallelism.

Containers in Memoria are **not** thread safe, and this is foundational design decision to make data structures simpler. All thread-safety, if any, are provided at the level of storage engines. And the main concurrency and parallelism mechanism Memoria relies on is [persistent/functional](https://en.wikipedia.org/wiki/Persistent_data_structure) data structures. This feature also comes with its design costs, limitations and overheads. But it also gives Memoria all of its batteries and superpowers.

So, B+Tree-based containers in Memoria can be of two implementation types:

1. **Copy-on-Write-based (CoW)** containers. Persistence is supported at the level of containers. This is the fastest option but at the expense of more complicated container design. All container types need to align with CoW semantics, that is well-encapsulated by the Framework.
2. **Epithemeral (non-CoW)** containers. This type of containers do not explicitly support CoW semantics themselves, so the *may* have parent and sibling links if necessary. But for the sake of code unification and reuse, they *don't*. CoW semantics may still be supported at the level of *storage engines*. For example, there is a variant of storage engine on top of the LMDB database that has strongly serialized CoW-based transactions. When working on top of such storage engines, containers do not need to provide their own CoW semantics. See more details on that in the [Storage engines](/docs/overview/storage/) section.





TBC...
