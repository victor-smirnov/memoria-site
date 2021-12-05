---
title: "Multiary Wavelet Trees"
description: ""
lead: ""
date: 2021-10-28T02:07:01-04:00
lastmod: 2021-10-28T02:07:01-04:00
draft: false
menu: 
  docs:
    parent: "datazoo"
weight: 1040
toc: true
---

Wavelet trees (WT) are data succinct rank/select dictionaries for large alphabets with many [practical applications](http://arxiv.org/abs/1011.4532). There is a good [explanation](http://alexbowe.com/wavelet-trees/) of what binary wavelet trees are and how they work. They provide rank() and select() over symbol sequences ($N$ symbols) drawn from arbitrary fixed-size alphabets ($K$ symbols) in $O(log(N) * log(K))$ operations, where logarithms are on the base of 2. Therefore, for large alphabets, $log(K)$ is quite a big value that leads to big hidden constants in practical implementations of the binary WT. 

In order to improve runtime efficiency of wavelet trees we have to lower this constant. And one of the way here is to use multiary cardinal trees instead of binary ones. In this case, for $M$-ary cardinal tree we will have $log(M)$ speedup factor over binary trees (tree height is $log(M)$-times smaller).

## Wavelet Tree Structure

Let we have a sequence of integers, say, 54.03.12.21.47.03.17.54.22.51 drawn from 6-bit alphabet. The following figure shows 4-ary wavelet tree for this sequence. Such WT has $6/log_2(4) = 3$ levels. 

First we need to represent our sequence in a different format. Our WT is 4-ary and has 3 layers. We need to "split" the sequence in 3 layers horizontally where symbols of each layer are drawn from 2-bit alphabet. In other words, we need to recode our sequence from base of 10 to base of 4, and then write numbers vertically:

 {{< figure src="multiary_wavelet_tree.svg" >}}

Note that this is just a logical operation, it doesn't require any transformation of the sequence itself.

Our WT is a 4-ary cardinal tree, each node has from 0 to 4 children. Each child represents one symbol from the layer's alphabet. Note that in general case it isn't necessary to draw all layers from the same alphabet, but it simplifies implementation.

In order to build WT, perform the following steps:

1. Assign top layer (Layer 2) of the sequence to the root node of WT.
2. For each symbol of Layer 1 put it to the subsequence of the node with the cardinal label matched with corresponding symbol from the same position in the Layer 2.
3. Repeat step (2) for symbols at Layer 0 but now select appropriate child at Level 1 of the tree, using pair of symbols from the same positions at Layer 1 and Layer 2 of the sequence.

Check the figure for details. Symbols in the WT and the sequence are colored to simplify understanding of symbols' distribution.

Note that for an alphabet with K symbols, multiary WT has up to K leafs that can be very significant number. But for most practical cases this number is moderate. The larger number of distinct symbols in the sequence, the bigger tree is. Dynamic LOUDS with associated cardinality labels is used to code structure of WT.

Also, it is not necessary to keep empty nodes in the tree (they are shown in gray on the figure).

## Insertion and Access

To insert a value into WT we need:

1. find the path from root to leaf for inserted values, insert tree nodes if necessary;
1. find correct position in the node's subsequence to insert current symbol.

The path in the wavelet tree is determined by "layered" representation if inserted symbol. Computation of insertion position is a bit tricky.

Let we insert the value of 37 into position 7. Layered representation of 37 is "211".

1. Level 2. Insert "2" into position 7 of root node's subsequence of WT.
1. Level 1. Next child is "2". Insertion position for "1" is `rank(7 + 1, 2) - 1 = rank(8, 2) - 1 = 1` computed in the parent node's sequence for this child.
1. Level 0. Next child is "1", create it. Repeat the procedure for Layer 1. Insertion position for "1" is `rank(1 + 1, 1) - 1 = rank(2, 1) - 1 = 0` computed in the parent node's sequence for this child.

See the following figure for details:

 {{< figure src="multiary_wavelet_tree_insert.svg" >}}


Access is similar, but instead of to insert a symbol to a node's subsequence, take the symbol form it and use it to select next child. 

## Rank

To compute rank(position, symbol) we need:

1. find the leaf in WT for the symbol;
1. find position in the leaf to compute the final rank.

 {{< figure src="multiary_wavelet_tree_rank.svg" >}}


## Select

Computation of select(rank, symbol) is different. If rank() is computed top-down, then select() is computed bottom-up.

Let we need to select position of the 2nd 3 in the original sequence. Layered representation for 3 is "003".

1. Find the leaf in WT for the given symbol.
2. Perform `select(2, "3") = Pos0` on the leaf's sequence.
3. Walk up to parent for his leaf. Perform `select(Pos0 + 1, "0") = Pos1`.
4. Step up to the parent node (the root). Perform `select(Pos1 + 1, "0") = Pos`. This is the final result.

Check the following figure for details:

 {{< figure src="multiary_wavelet_tree_select.svg" >}}

## Implementations

In Memoria, Multiary wavelet tree consists of four distinct data structures.

1. LOUDS to store wavelet tree structure.
2. Tree-ordered sequence of cardinal labels for tree nodes.
3. Tree-ordered sequence of sizes for tree node's sub-sequences.
4. Tree-ordered sequence of node's symbols.

The first three structures are implemented as single [Labeled Tree](/docs/data-zoo/louds-tree) with two labels. The first one is cardinality of the node in its parent. The second one is size of node's subsequence. 

The fourth data structure is a separate [Searchable Sequence](/docs/data-zoo/searchable-seq) for small sized alphabets.

Memoria has two different implementations of WT algorithm. The first one is dynamic WT that provides access/insert/select/rank operations performing in O(log _N_) time. 

The second one has all those four data structures implemented with [Packed Allocator](Memory_Allocation) placed in a single raw memory block of limited size. This implementation has fast access/select/rank operations but slow insert operation with O(_N_) time complexity. 

Currently Memoria provides only 256-ary wavelet tree for 32-bit sequences. Other configurations will be provided in upcoming releases of the framework.
