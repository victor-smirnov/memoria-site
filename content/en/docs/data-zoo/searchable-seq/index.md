---
title: "Searchable Sequence"
description: ""
lead: ""
date: 2021-10-28T02:07:01-04:00
lastmod: 2021-10-28T02:07:01-04:00
draft: false
menu: 
  docs:
    parent: "datazoo"
weight: 1020
toc: true
---

Searchable sequence or rank/select dictionary is a sequence of symbols that supports two operations:

1. `rank(position, symbol)` is number of occurrences of `symbol` in the sequence in the range [0, position)
2. `select(rank, symbol)` is a position of `rank`-th occurrence of the `symbol` in the sequence.

Usually the alphabet is {0, 1} because of its practical importance, but larger alphabets are of significant interest too. Especially in Bioinformatics and Artificial Intelligence.

There are many implementations of binary searchable sequences (bitmaps) providing fast query operations with $O(1)$ time complexity. Memoria uses partial sum indexes to speedup rank/select queries. They are asymptotically slower than other methods but have additional space overhead for the index. 

Packed searchable sequence is a searchable sequences that has all its data structured packed into a single contiguous memory block with packed allocator. It consists from two data structures:

1. multi-index partial sum tree to speedup rank/select queries;
2. array of sequence's symbols.

See the following figure for the case of searchable bitmap.

 {{< figure src="packed_seq.svg" >}}

Note that for a sequence with K-symbol alphabet, packed sum tree has K indexes, that results in significant overhead even for relatively small alphabets. For example 8-bit sequence has 256-index packed tree that takes more than 200% of raw data size if the tree is not compressed. To lower this overhead Memoria provides various compressed encodings for the index's values.

## Creation and Access

To create partial sum tree for a sequence we first need to split it logically into blocks of fixed number of symbols (16 at the figure). Then sum different symbols in the block, each such vector is a simple partial sum tree leaf. Build other levels of the tree accordingly.

* Symbol update is relatively fast, it takes $O(log(N))$ time.
* Symbol insertion is $O(N)$, it requires full rebuilding of partial sum tree.
* Symbol access does not require the tree to perform, it takes $O(1)$ time.

## Rank

To compute `rank(position, symbol)`

1. Given `position`, determine sequence `block_number` for that position, and `block_pos` position in the block, $O(1)$;
2. `block_rank` = [sum](/docs/data-zoo/partial-sum-tree)(0, `block_number`) in the sum tree, $O(log(N))$;
3. count number of `symbol`s in the block to `block_pos`, $O(1)$;
4. final rank is (2) + (3).

## Select

To compute `select(rank, symbol)`

1. Given partial sum tree and a `symbol`, determine target sequence `block_number` having `total_rank <= rank` for the given `symbol` using [findLE](/docs/data-zoo/partial-sum-tree) operation, $O(log(N))$;
2. For the given block, compute `rank_prefix` = [sum](/docs/data-zoo/partial-sum-tree)(0, `block_number`) for the given `symbol`, $O(log(N))$;
3. Compute `local_rank = rank - rank_prefix`;
4. Scan the block and find position of `symbol` having rank in the block = `local_rank`, $O(1)$.

Actual implementation joins operations (1) and (2) into a single traverse of the sum tree.

## Hardware Acceleration

Consideration are the same as for [partial/prefix sum trees](/docs/data-zoo/partial-sum-tree), especially, because searchable sequence contains partial sum tree as an indexing structure. 

Modern CPUs usually have direct implementations for rank (PopCount) for binary alphabets. Select operation may also be partially supported. To accelerate searchable sequences, it's necessary to implement rang/select over arbitrary alphabets (1-8 bits per symbol).

Symbol blocks are also contiguous in memory and can be multiple of DRAM memory blocks. Rank/select machinery is simpler or comparable with machinery for addition and subtraction. Those operations can be efficiently implemented in a small silicon budget and at high frequency.

