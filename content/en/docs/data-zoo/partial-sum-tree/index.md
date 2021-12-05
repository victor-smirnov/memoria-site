---
title: "Partial Sums Tree"
description: ""
lead: ""
date: 2021-10-28T02:07:01-04:00
lastmod: 2021-10-28T02:07:01-04:00
draft: false
menu: 
  docs:
    parent: "datazoo"
weight: 1010
toc: true
---

## Description

Let's take a sequence of monotonically increasing numbers, and get delta sequence from it, as it is shown on the following figure. Partial Sum Tree is a tree of sums over this delta sequence.

 {{< figure src="trees.svg" >}}

Packed tree in Memoria is a multiary balanced tree mapped to an array. For instance in a level order as it shown on the figure.

Given a sequence of N numbers with monotonically increasing values, partial sum tree provides several important operations:

* `findLT(value)` finds position of maximal element less than `value`, time complexity $T = O(log(N))$
* `findLE(value)` finds position of maximal element less than or equals to `value`, $T = O(log(N))$.
* `sum(to)` computes plain sum of values in the delta sequence in range [0, to), $T = O(log(N))$
* `add(from, value)` adds `value` to all elements of original sequence in the range of [from, N), $T = O(log(N))$.

It is obvious that first two operations can be computed with binary search without partial sum tree and all that overhead it introduces. 

## Hardware Acceleration

Partial or prefix sum trees of higher degrees are especially suitable for hardware acceleration. DDR and HBM memory transfer data in batches and works best if data is processed also in batches. The idea here is to offload tree traversal operation from CPU to the memory controller or even to DRAM memory chips. Sum and compare operations are relatively cheap to implement, and no complex control is required. 

By offloading tree traversal to the memory controller (that usually works in front of caches), we can save precious cache space for more important data. By offloading summing and comparison to DRAM chips, we can better exploit internal memory parallelism and save memory bandwidth. In such distributed architecture, a single tree level scan can be performed with the latency and in the power budget of a _single random memory access_, saving energy and silicon for other computations.

