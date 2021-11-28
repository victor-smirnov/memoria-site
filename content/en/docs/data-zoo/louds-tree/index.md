---
title: "Level Order Unary Degree Sequence (LOUDS) "
description: ""
lead: ""
date: 2021-10-28T02:07:01-04:00
lastmod: 2021-10-28T02:07:01-04:00
draft: false
menu: 
  docs:
    parent: "datazoo"
weight: 1030
toc: true
---

Level Order Unary Degree Sequence or LOUDS is a special form of ordered tree encoding. To get it we first need to enumerate all nodes of a tree in level order as it is shown of the following figure.

![LOUDS Tree Encoding](louds.png)

Then for each node we write its degree in unitary encoding. For example, degree of the fist node is 3, then its unitary encoding is '1110'. To finish LOUDS string we need to prepend substring '10' to it as it is shown on the figure. Given an ordered tree on N nodes LOUDS takes no more than 2N + 1 bits. This is very succinct implicit data structure.

LOUDS is a bit vector. We also need the following operations on it:

* `rank1(i)` -- returns number of '1' in the range [0, i)
* `rank0(i)` -- returns number of '0' in the range [0, i)
* `select1(rnk)` -- returns position of rnk-th '1' in the LOUDS string, rnk = 1, 2, 3, ...
* `select0(rnk)` -- returns position of rnk-th '0' in the LOUDS string, rnk = 1, 2, 3, ...

Different ways of tree node numbering for LOUDS are possible, Memoria uses the simplest one. Tree node positions are coded by '1'. 

* `node_num = rank1(i + 1)` -- gets tree node number at position `i`;
* `i = select1(node_num)` -- finds position of a node in LOUDS given its number in the tree.

Having this node numbering we can define the following tree navigation operations:

* `fist_child(i) = select0(rank1(i + 1)) + 1` -- finds position of the first child for node at the position `i`;
* `last_child(i) = select0(rank1(i + 1) + 1) - 1` -- finds position of the last child for node at the position `i`;
* `parent(i) = select1(rank0(i + 1))` -- finds position of the parent for the node at the position `i`;
* `children(i) = last_child(i) - first_child(i)` -- return number of children for node at the position `i`;
* `child(i, num) = first_child(i) + num` -- returns position of num-th child for the node at the position `i`, `num >= 0`;
* `is_node(i) = LOUDS[i] == 1 ? true : false` -- checks if `i`-th position in tree node.

Note that navigation operations only defined for positions `i` for those `is_leaf(i) == true`.

### Example

Let we find number of the first child for the node 8. 

1. `select1(8) = 11`;
2. `first_child(11) = select0(rank1(11 + 1)) + 1 = select0(8) + 1 = 19`;
3. `rank1(19 + 1) = 12`.

The first child for node 8 is node 12.

The following figure shows how the parent() operation works:
![LOUDS Tree parent() navigation](louds-parent.png)

### Limitations

Node numbers are valid only until insertion (or deletion) into the tree. Additional data structure is necessary to keep track of node positions after tree structure updates.

## Labeled Tree

Labeled tree is a LOUDS tree with a fixed set of numbers (or 'labels') associated with each node. It is implemented as multistream balanced tree where the first stream is dynamic bit vector with rank/select support, and the rest are streams for each label.

Streams elements distribution is very simple for this data structures. Each label belongs to a tree node that is coded by position of '1' in LOUDS. Each leaf of balanced tree has some subsequence of the LOUDS. 

![LabeledTree Leaf Node Layout](labeled_tree_leaf.png)

Let we have a LOUDS sub-stream of size N with M 1s for a given leaf. Then this leaf must contain M labels in each label stream. The following expressions links together node and level positions withing a leaf:

* `label_idx = rank1(node_idx + 1) - 1`;
* `node_idx = select1(label_idx + 1)`;

Where `label_idx` is in [0, M) and `node_idx` is in [0, N)

LabeledTree uses balanced partial sum tree as a basic data structure. Because of that it optionally supports partial sums of labels is specified tree node range. The most interesting range is `[0, node_idx)`. See [Multiary Wavelet Tree](/docs/data-zoo/wavelet-tree) for details.
