---
title: "Associative Memory (Part 1)"
date: 2021-10-28T02:07:01-04:00
lastmod: 2021-10-28T02:07:01-04:00
draft: false
menu: 
  docs:
    parent: "datazoo"
weight: 1070
toc: true
---

## What is Associative Memory

Associative memory is content-addressable memory, where the item is being addressed given some part of it. In a broad sense, associative memory is a model for high-level mental function of _Memory_. Such level of complexity is by no means a simple thing for implementation, though artificial neural networks have demonstrated pretty impressive results (at scale). In this article we are scaling things down to the level of bits and showing how to design and implement content-addressable memory at the level of _bits_.

## Motivating Example: RDF

In [Resource Description Framework](https://en.wikipedia.org/wiki/Resource_Description_Framework) data is modelled with a special form of a labelled graph, consisting from _facts_ (represented as URIs) and _triples_ in a form of $(Subject, Predicate, Object)$ linking various facts together. Logical representation of this _semantic graph_ is a table, enumerating all the triples in the graph. The main operation on the graph is _pattern matching_ using SQL-like query language [SPARQL](https://en.wikipedia.org/wiki/SPARQL). Another common mode of operations over graphs is traversal, but this mode is secondary for semantic graphs. Pattern-matching in semantic graphs is based of _self-joins_ over the triple tables:

 {{< figure src="triples.svg" >}}

Because of the flexibility of SQPRQL, self-joins can be performed on any combination of Subject, Predicate and Objects, and the join on objects is the main reason why basic representation of graphs for RDF is a relational table. And, if we want fast execution, this table has to be properly indexed. 

The simplest way to provide indexing over a triple table is to sort it in some order, but ordered table only allows ordered _composite_ keys. If a table is ordered like (S, P, O), that means $(Subject, Predicate, Object)$, _then_ we can search first by Subject, _then_ buy Predicate, and only then by Object. But not vise versa. If we want to search by an Object first, we need another table ordered by object: (O, X, Y). To be able to search in any order we need all possible permutations of S, P and O: it's $3! = 6$ tables. 

So, fully indexed RDF triple store will need at least 6 triple tables. How many is it? 

1. It's _up to_ 6 times the size of the original single-table store (not counting URIs and Objects if they are stored separately). 
1. It's _up to_ 6 times slower insertions if they are not paralleled. Of course we can make many insertions in parallel, but it's _6 times more energy_ anyway.

Modern triple stores model semantic graphs with _quads_, adding _Graph ID_ to a triple, so nodes can link to entire graphs, no just other nodes (self-referentiality). In addition to, higher-dimensional tables ($D > 4$) can also be provided to speedup certain queries. And, in general case, if we have $D$-dimensional data table, we need _up to_ $D!$ orderings of this table. That is 24 for $D=4$ and grows faster than an exponent.

Sorting relational tables does not scale at all for higher-order graphs ($D > 3$), but we can do better.

## Definition of Associative Memory

So, without loss of generality, associative memory is a $D$-dimensional relation $R$ with _set_ semantics, over integer numbers drawn from some finite set (domain). The main operation on the memory is _lookup_ that can be performed using arbitrary number of dimensions, specifying _match_, _range_ or _point_ lookup, or any combination of for any number of dimensions. _Recall_ is the result of _lookup_ operation, and is enumeration of all entries in $R$ matching the query.

Associative memory can be either _static_, if only lookups are allowed. Or _dynamic_, if it supports insert, update and delete operation for individual elements (Update operation can be reduced to delete + insert). 

## Multiscale Decomposition

Let we have a $D$-dimensional relation $R = \lbrace {r_0, r_1, r_2, ..., r_{N-1}}\rbrace$, representing a $set$, where $r_i = (c_0, c_1, ..., c_{D-1})_i$ - а $D$-dimensional tuple and $N$ is a 'size' of the table (number of elements in the set). $c_{i,j}$ is a table's cell value from row $i$ and dimension (column) $j$. Each cell value $c_{i,j}$ has a domain of $H$ bits.

Example: A set of 8x8 images with 8 bits per pixel can be represented with 64-dimensional relation with $H = 8$. Maximal number of images in a set is $N <= 8^{64} = 2^{192}$. Given such table we can easily define an associative memory reducing content-addressable lookup to linear table scans and bit manipulations (time complexity is $O(N)$). And, actually, this is how it's implemented for approximate nearest neighbour search on massively-parallel hardware. Parallel linear scan is fast, but it's not scalable (fast memory is expensive) and it's not energy-efficient. 

Fortunately, we can transform $O(N)$ into $O(P H + M)$ _"on average"_, where $1 <= P <= 2^D$ -- average number of nodes per **bucket** (see below), that is, thanks to the ["Blessing of Dimensionality"](https://en.wikipedia.org/wiki/Curse_of_dimensionality#Blessing_of_dimensionality), usually tends to 1. And $M$ is a _recall size_, number of points returned by the query over $R$.

To perform the multiscale decomposition of relation $R$, we need to perform the following transformation for each row $r_i$:

Let $c_{ij} = S_{ij} = (s_0, s_1,s_2,...,s_{H-1})_{ij}$, where $s \in \lbrace{0, 1}\rbrace$ is a bit-string representation of cell value $c$. Let $s_h = B(S, h)$ -- $h$-th bit of string $S$.

Now, $M(r)$ is a multiscale decomposition of a row $r = (S_0, S_2, ..., S_{D-1})$. Informally, $M(r)$ is a bit string consisting from a concatenation of shuffling of all bits form $S_j$:

$M(r) = B(S_0, H-1) B(S_1, H-1) ... B(S_{D-1}, H-1)| ...B(S_0, H-1) B(S_1, H-1) ... B(S_{D-1}, H-1)| ... B(S_0, 0) B(S_1, 0) ... B(S_{D-1}, 0)$. 

The symbol $|$ is added to graphically separate $H$ _layers_ of the multiscale representation from each other.

Example. Let $r = (100, 110, 001)$. Then $M(r) = 111|010|001$. Note, that in some sense, $M(r)$ is producing a point on a [$Z$-order curve](https://en.wikipedia.org/wiki/Z-order_curve) for $r$. This correspondence may help in some applications.

So, multiscale decomposition of $T = M(R)$ converts a table with $D$ columns into a table with $H$ columns, which are called _layers_ here. 

Now, let's assume, that the table $T$ is sorted in a bit-lexicographic order.

 {{< figure src="tables.svg" >}}
 
What is special about table $T$ is that every column $L_j$ contains some information form each column $D_j$ from table $R$. So, by searching in a column of $T$ we can search in all columns from $R$ at once. Table $T$ itself does not provide any speedup over the sorted $R$ because the number of rows is still the same as for table $R$. But quick look at the table will show that there are many _repetitions_. So, we can transform $T$ into a tree, by hierarchically (here, from left to right) collapsing repetitive elements in tables' columns. Now, a **bucket** is a list of all children of the same parent node sorted lexicographically. It can be shown, that there may be _at most_ $2^D$ elements in a bucket. So, search in such data structure is $O(2^D H + M)$ _"on average"_. If $D$ is small, say, 16 or less, this may dramatically improve performance relative to linear scan of $R$ (even if it's sorted). See the [Analysis](#analysis) section below for additional properties and limitations.

Note that each path from root to leaf in the tree encodes a single row in the table $T$, and after the inverse multiscale decomposition, in $R$.

Let's demonstrate how search works. Let we want to enumerate all rows in $R$ with $D_2$ = 0111, so $Q = (X, Y, 0111)$, where $X$ and $Y$ are _placehoders_. First, we need a multiscale representation of $Q$, $M(Q) = xy0|xy1|xy1|xy1$. Now, we need to traverse the tree from root to leafs, according to this pattern:

 {{< figure src="search.svg" >}}

Having the multiscale query encoding, the tree traversal is straightforward. We are visiting sub-trees in-order, providing that current pattern matches the node's label. Visiting the leaf (+ its label is matched) means full match. The excess number of nodes visited by a range query is called _overwork_.

In this example, the selectivity of the query is pretty good. All leafs matching the query are being visited (visited nodes are drawn in the dotted-green line style). Note the nodes marked with (*). The traversal process visited some nodes _not leading to the full match_. This is why type complexity estimation for this data structure is logarithmic _"on average"_. Its performance depends on how data is distributed in the tree. 

## Using LOUDS for the Tree

Table $T$ has the same size (in bits) as the table $R$, but the tree, if implemented using pointer-based structure, will add a lot to the table representation. Fortunately, we can use [LOUDS Tree](/docs/data-zoo/louds-tree) to encode the tree. LOUDS tree has very small memory footprint, only 2 bits per node (+ a small overhead, like 10%, for rank/select indices). Search tree size estimation is at most $NH$ nodes, so the tree itself will take at most size of two columns of the table $R$ ($2NH$ + small overhead). In most cases LOUDS-encoded $T$ will take less space that original table $R$, including auxiliary data structures.

What is the most important for LOUDS Tree is that it's structured in memory in a linear order. So, if for some reason a spatial tree traversal degrades into linear search, the tree will be pretty good at this. We just need to read many layers of the tree in parallel. Such I/O operations can be efficiently prefetched.

## Analysis

It can be shown that the tree is very similar to a trie-based Quad Tree (for high dimensions), so most expected (average-case) and worst-case estimations also apply. In worst case, for high-dimensional trees, traversal degenerates into linear a search. Fortunately for LOUDS, it's both memory-efficient for linear search _and_ for tree traversal. But on average, overwork is moderate, if doesn't even tend to zero. And queries perform pretty well.

Let's return to RDF triple stores and compare things to each other. Let's assume that $D = 3$ (number of dimensions) and $H = 32$ (resource identified size or search tree depth). So, in case of insertion of a triple, we have to perform 6 insertions into 6 triple tables (for each ordering) and 32 insertions in case of the tree (into each tree level). Reading is also slower: one lookup in a sorted table vs 32 lookups in the tree. It looks unreasonable to switch from triple tables (worst case logarithmic) to search tree (average case logarithmic), unless we are limited in memory and want to fit as many triples as possible into the available amount. But things start changing when we go into higher dimension ($D > 3$) and need more indices to speedup our queries.

So far...

1. Point-like operations (to check if some row exists in the relation $R$) will take $O(D log(N))$ time.

1. Range search in the quad trees is logarithmic on average, but it's relatively easy to build a worst-case example, when performance degrades to a linear scan. For example, if $D = 64$, maximal bucket size is $2^{64}$, that is much larger than any practical $N$ (number of entries in $R$). Unless the data is distributed uniformly among _different levels_ of the tree, we will end up having a few but very big buckets. So, special care must be taken on how we map our high-level data to dimensions of the tree. Random mapping is usually a safe bet, but always the best choice.

1. In the search tree "Blessing of Dimensionality" is fighting with "Curse of Dimensionality", it's kind of $0 \cdot \infty$. In higher dimensions data tends to be extremely _sparse_ because volume size grows exponentially with number of dimensions. So, normally, even in high dimensions, buckets will tend to have small number of elements. The bigger the number -- the better, because it improves data compression and speeds up queries. But beware of the worst case, when the tree has one big bucket that all queries are visiting. It has also been observed for similar K-d trees, that with higher number of dimensions, _overwork_ also tens to increase (the blessing vs curse situation, $0 \cdot \infty$, who wins?). 

1. In case if LOUDS tree is dynamic and implemented using B-tree, _insert_, _update_ and _delete_ operations to the search tree have $O(H + log(N))$ time complexity. 

1. The data structure representation in memory is very compact. It's the size of original table $R$ + (up to) the size of two columns from $R$ for LOUDS tree + 5-10% of the tree to auxiliary data structures. Overhead of the tree is constant and is amortizing with higher number of dimensions.

1. Even if we work with high-dimensional data, and we are losing to the curse of dimensionality, it's possible to perform approximate queries. Many applications where high-dimensional data analysis is required, like AI, are essentially approximate. LOUDS tree allows to compactly and efficiently remember additional bits of information with each node, to facilitate approximate queries (if necessary).

## Hardware Acceleration

LOUDS tree-based Associative memory seems to be impractical specifically for RDF triple stores, but if hardware accelerated, can be cost-effective at scale, providing also many other benefits, not just memory savings (which are huge for higher dimensions). The bottleneck is on the update operations, where insertion and deletion may require tens of B-tree updates. Fortunately, this operation is well-parallelizable so we can use thousands of small RISC-V cores equipped with special command for direct and energy-efficient implementation of essential operations (partial/prefix sums, rank and select). An array or cluster of such cores can even be embedded into [storage memory controller](/subprojects/smart-storage).
