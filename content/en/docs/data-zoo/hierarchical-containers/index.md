---
title: "Hierarchical Containers"
description: ""
lead: ""
date: 2021-10-28T02:07:01-04:00
lastmod: 2021-10-28T02:07:01-04:00
draft: false
menu: 
  docs:
    parent: "datazoo"
weight: 1027
toc: true
---

In this page it's explained how various hierarchical containers (vectors with large values, multimaps, tables, wide tables, etc) are organized inside and how [searchable sequences](/docs/data-zoo/searchable-seq/) make it possible.

## Multimap

Let, for simplicity, we have a multimap `std::map<int64_t, std::vector<uint8_t>>` (we call it a *Multimap* here) that is internally a binary search tree (RB-/AVL-tree etc) with vectors as values. Vector<T> is internally a contiguous data structure but the search tree is basically a tree-shaped linked list or *pointer chasing* data structure. This data layout works perfectly for the *word*-addressed main memory, but if we want to place out data structure into *block*-addressed external memory, the whole thing gets trickier, because the using of allocation now is pretty large comparing to RAM: 4KB and greater. In the external memory we can still combine Map (represented as B+Tree) with Dynamic Vector (represented as B+Tree) the same way how two those containers `std::map<>` and `std::vector<>` are combined with each other (via references) and get either a set of B+Trees or one 'hierarchical' B+Tree, but the minimal size of value vector will be *one block* that is pretty large. Practical applications can optimize this specific case using various techniques like placing short vectors right inside the parent container's (Map in this case) leaf block, and only spilling a new B+Tree when there is no more room for that in the block. It works pretty well in practice, but we can do much better. Below it's explained how we can use searchable sequences for fitting arbitrary-shaped hierarchical containers into a single (or 'flat') B+Tree.

### Data structure

Let for certainty we have the following Multimap: `{1=[1,5,2,4], 4=[7,3,1], 6=[5,9,1,2,0], 7=[8,1]}`. We represent is with *three* arrays: keys `K[]`, values `V[]` and [searchable symbol sequence](/docs/data-zoo/searchable-seq/) `S[]`. `Srle[]` is an [RLE encoding](/docs/data-zoo/compressed-symbol-seq/) for `S[]` (we need one `S[]` or `Srle[]`, *not* both):

 {{< figure src="multimap.svg" >}}

The rule is simple. The keys vector `K[]` should be built in the increasing key order to use binary search for fast lookup. The values vector `V[]` should be built in the *key vector's order* by concatenating corresponding vectors.

The symbol sequence `S[]` is constructed in the following way. For each key in key's vector `K[]` we put `0` and for each value element in the *associated value's sub-vector* `V[]` we put `1` as it's shown at the picture above. We will be using [`rank()` and `select()`](/docs/data-zoo/searchable-seq/) operations for implementing access and update operations over our Multimap.

To find a value for a key we first need to find a position for this key in the sorted vector `K[]` by using binary search: `Px = bsearch(S[], Kx)`. Given the position `Px` for key `Kx` we can find corresponding position `Sx` in the `S[]` by using the `select` operation: `Sx = select(S[], Px + 1, 0)` -- we are looking for position of `Px`-th `0` in `S[]`. `Sx + 1` will be position of the first symbol `1` in `S[]`, corresponding to the first value, associated with `Kx`. In order to locate related position in `V[]` we need to calculate total size of all sub-vectors for keys *before* `Kx`: `Vx = rank(S[], Px, 1)`. In order to find number of elements in specific sub-vector, associated with `Kx` we can use `count()` operation -- count number of symbols in a run, starting from the specified one: `Lx = count(S[], Px + 1, 1)` -- counting number of `1` in `S[]` starting from `Px + 1` till we either hit the next `0` (next key's symbol mark in the sequence) or the end of the sequence.

By applying the same math we can derive corresponding operations for updating the Multimap structure: insert entries, remove entries, merge and split maps and so on. 'Rank()', 'select()' and 'count()' have logarithmic time complexity.

Spending one bit for every value element in the Multimap is not necessary. We can easily use RLE encoding `Srle[]` for `S[]` as it's shown at the picture above. For large value sub-vectors (file system/object store) space saving may be significant.

### Representing relational tables

Note that Multimap is sufficient for representing a row-wise *clustered* relational table. `K` is a table's *primary key* and corresponding sub-vector may contain row's content in an unstructured form. If we want unclustered (regular) table without a primary key, we just not needed the `K[]` vector: `std::vector<std::vector<V>>`. Everything else is the same. 

Such table representation has two notable properties:

1. To *scan* a table we just need to scan three perfectly memory-aligned data structures concurrently. 

2. Table may easily have *very large* rows, as well very small rows. There is no any intrinsic memory overhead for this.

## Multimap with searchable values

Practically important case is `std::map<K, std::set<V>>` or, here, SearchableMultimap used, for instance, for implementing sparse graphs. If `K` and `V` are graph node's identifiers, SearchableMultimap may be used for storing node's neighbours in the graph. 

The easiest way to make sub-vectors efficiently searchable is just to sort them. And this is the only option if identifiers are not numbers. Because we know the size of sub-vector for a given key, we can binary-search in this sub-vector only.

The second option is to use the same technique that is used for [partial sum trees](/docs/data-zoo/partial-sum-tree/). If identifiers are numbers supporting `+` and `-` binary operations, we can build *delta sequences* for each sub-array individually and concatenate them into `V[]`. Unlike locally-sorted sub-vectors, such delta-sequence is globally-searchable. We just need to add *prefix* `Nx = sum(V[], Vx)` for key the `Kx` to the `Kx`'s value.

## Wide Table

*Wide Table* is a data structure of the form `std::map<RKT, std::map<CKT, std::vector<V>>>` where `RKT` is a row key type and `CKT` is a column key type. Each row of a Wide Table may have arbitrary, practically unlimited, number of columns. This is rarely needed in practice for *relational* tables. But may be needed for other *application-level* data structures built on top of wide tables. Basically, wide table is a sparse matrix or a sparse graph.

 {{< figure src="wide-table.svg" >}}

It's rather easy to get wide table from a Multimap, we just need an alphabet with three symbols instead of two: `0` means row row key RK, `1` means column key CK and `2` means column data. Everything else, including navigational operations are basically the same.

## Generalized Hierarchical Container

The pattern above can be generalized. Single-level containers, like Map or Vector need a symbol sequence with zero-size alphabet (or no sequence at all). Each new layer add one symbol to the alphabet. Memoria relies heavily on this property for complex containers. For an L-level container we have L vectors and a searchable symbol sequence (so, L + 1 vectors total). 

## Multistream B+Tree.

There are basically two strategies of implementing generalized hierarchical containers for external memory:

1. *One B+Tree per vector or sequence*. This is the easiest option because we just need two separately engineered B+Trees. There are two main drawbacks here. First, small structures may take up to `L + 1` blocks of memory. Second, for each query we need to search in multiple B+Trees (from root down to leafs). 
2. *Combine all vectors in one B+Tree*, enforcing locality principle: data accessed together shout be in the same block or very close to each other. It can make certain important queries faster, but by the expense of other types of queries.

No strategy is universally the best, but the second one is better if we want to optimize things for *reading*. Memoria may use them both for different reasons, but it primarily relies on the second one -- *multistream B+Tree*.

The basic idea is simple. 

If we are representing a Multimap with a multistream B+Tree, we need to split `Srle[]`, `K[]` and `V[]` in such way that all related array elements be put into the same leaf node. In this case, for example, if we found specific key, associated values will be either in this leaf (most likely) or in the next one (pretty quick operation).

Branch node's structure is similar. We need one array for maximum keys for each child (Key Stream), and two arrays for sums of `0` and `1` in the subtree's `Srle` sequence (Srle Stream). And, of course, child node ID array:

{{< figure src="multistream_tree_nodes.svg" >}}

Branch nodes for `Srle` sequence form a [partial/prefix sum tree](/docs/data-zoo/partial-sum-tree/), that can be used for efficient implementation of 'rank()' and 'select()' operations.

Keys stream in this specific case may be searched in a usual way for max-type B+tree.

Note that in this example branch nodes do not show Value Stream, because values are not searchable, so we don't need to store separate *index* info for values. 

Note also that mutistream B+Tree implementation in Memoria is slightly different in the way how streams are represented in tree blocks, here we omitted some details for brevity.
