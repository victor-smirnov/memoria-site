---
title: "Mutistream Balanced Tree"
description: ""
lead: ""
date: 2021-10-28T02:07:01-04:00
lastmod: 2021-10-28T02:07:01-04:00
draft: true
menu: 
  docs:
    parent: "datazoo"
weight: 1050
toc: true
---


## The Problem 

Let us consider data structure mapping an integer `ID` to a dynamic vector. Nothing complicated is here but there is one requirement: it has to represent small data values compactly, without that overhead most file systems have. That means we can't just use `Map<ID, Vector<>>` because like any other container, even empty `Vector<>` consumes at least one memory block. So this overhead will be very significant for large number of small data values.

The solution is to store all data values in a single dynamic vector and use an additional data structure for the dictionary of data values. This dictionary is a set of pairs in the form `<ID, DataOffset>`.

 {{< figure src="vector_map.svg" >}}

The dictionary can be easily represented via a balanced partial sum tree having two indexes, one for `ID` and one for `DataOffset` as it is shown on the following figure: 

 {{< figure src="double_index.svg" >}}

An multi-index partial sums tree is just an ordinary [partial sum tree](/docs/data-zoo/partial-sum-tree) except that scalar values of sums regarded as vectors.

There is no overhead for small data values in this model because both data structures (the dictionary and the dynamic vector) are represented in the most compact way. But now there is performance overhead because each data value search requires traversal of two balanced trees, the dictionary and the dynamic vector.

## The Solution

The solution is to intermix the dictionary and the data within a single balanced search tree so that the dictionary entries are placed closer to its data. 

Let us think that a balanced tree of a dictionary entry is the primary one. If all data values are small we can put all of them into the same tree leafs with their dictionary entry. But in general case data entries can be much larger than limited capacity of the leaf page. To maintain large data values we need complete dynamic vector sub-structure in the balanced tree of dictionary entries.

The following figure shows how it can be done:

 {{< figure src="multistream_tree_nodes.svg" >}}

In the leaf nodes of balanced tree we put two dynamic array: 

1. an array of dictionary entries (red, blue) and
2. an array of data (green). 

In the branch nodes we put three dynamic arrays: 

1. the array of partial sums for dictionary entries (red, blue), 
2. the array of partial sums for data lengths (green),
3. the array of child node IDs.

This joined balanced partial sum tree contains all information from both source trees, dictionary and dynamic entry ones. It is not necessary to place dictionary entries in the same leafs with their data. 

Then total balanced tree structure will look like this: 

 {{< figure src="multistream_tree.svg" >}}

It can be shown that entries can be placed in any place relative to their data providing that sequences of entries is ordered correctly. But logically linked entities should be placed closer to each other in terms of leaf nodes for performance reasons. 
