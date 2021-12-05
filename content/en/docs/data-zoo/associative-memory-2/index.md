---
title: "Associative Memory (Part 2)"
description: ""
lead: ""
date: 2021-10-28T02:07:01-04:00
lastmod: 2021-10-28T02:07:01-04:00
draft: false
menu: 
  docs:
    parent: "datazoo"
weight: 1080
toc: true
---

In the [previous part](/docs/data-zoo/associative-memory-1) we saw how LOUDS tree can be used for generic compact and efficient associative associative memory over arbitrary number of dimensions. In this part we will see how LOUDS trees can be used for function approximation and inversion.

## Definitions

Let $[0, 1] \in R$ is the domain and range we are operating on. $D$ is a number of dimensions of our space, and for the sake of visualizability, $D = 2$.

Let $\vec{X}$ is a vector encoding a point in $[0, 1]^D$, $\vec{X} = <x_0, x_1, ..., x_{D-1}>$. Let $str(x [, h])$ is a function converting a real number from $[0, 1]$ into a binary string by taking a binary representation of a real number to a form like "$0.0010111010...$", and removing leading "$0.$" and trailing "$0...$". so, $str(0.181640625) = 001011101$. If $h$ argument is specified for $str(\cdot, \cdot)$, then resulting string is either trimmed to $h$ binary digits, if it's longer than.

Let $len(x)$ is a number of digits in the result of $str(x)$. Note that $len(\pi / 10) = \infty$, so irrational numbers are literally infinite in this notation. Let $H$ is a maximal _depth_ of data, and there is some _implicitly assumed_ arbitrary value for $h$, like 32 or 64, or even 128. So we can work with _approximations_ of irrational and transcendent numbers, or with long rational numbers in a same way and without loss of generality.

Let $len(\vec{X}) = max_{\substack{i \in \lbrace 0,...,D-1 \rbrace }}(len(x_i))$.

Let $str(\vec{X}) = (str(x_0),..., str(x_{D-1}))$ is a string representation (a tuple) of $\vec{X}$. And let we assume, elements of the tuple are implicitly extended with '$0$' from the right, if their length is less than $len(\vec{X})$. In other words, all elements (binary string) of a tuple are implicitly of the same length.

Let $str(\lbrace \vec{X_0}, \vec{X_1}, ... \rbrace) = \lbrace str(\vec{X_0}), str(\vec{X_1}), ... \rbrace$. String representation of set of vectors is a set of string representation of individual vectors.

Let $M(\vec{X}) = M(str(\vec{X}))$ is a [multiscale transformation](/docs/data-zoo/associative-memory-1/#multiscale-decomposition) of binary string representation of $\vec{X}$. Informally, to compute $M(\vec{X})$ we need to take all strings from its string representation (the tuple of binary strings) and produce another string of length $len(\vec{X})  D$ by concatenating interleaved bits from each binary string in the tuple.

Example. Let $H = 3$ and $D = 2$. $\vec{X} = <0.625, 0.25>$, $str(\vec{X}) = (101, 01)$ and $M(\vec{X}) = 10|01|10$. Note that signs $|$ are added to separate $H$ _path elements_ in the recording, they are here for the sake of visualization and are not a part of the representation. The string $10|01|10$ is also called _path expression_ because it's a unique path in the multidimensional space decomposition _encoding_ the position of $\vec{X}$ in this decomposition. Visually:

 {{< figure src="multiscale1.svg" >}}
 
 Note that the order in which _path elements_ of a path expression enumerate the volume is [Z-order](https://en.wikipedia.org/wiki/Z-order_curve). 
 
Given a _set_ of $D$-dimensional vectors $\lbrace \vec{X} \rbrace$, it's multiscale transformation can be represented as a $D$-dimensional [Quad Tree](https://en.wikipedia.org/wiki/Quadtree). Such quad tree can be represented with a [cardinal LOUDS tree](/docs/data-zoo/louds-tree/#cardinal-trees) of degree $2^D$. Here, implicit parameter $H$ is a _maximal height_ of the Quad Tree. 
 
## Basic Asymptotic Complexity 
 
Given that $N = |\lbrace \vec{X} \rbrace|$, and given that LOUDS tree is dynamic (represented internally as a b-tree), the following complexity estimations apply:

1. Insertion and deletion of a point is $O(H log(N))$. Update is semantically not defined because it's a _set_. Batch updates in Z-order are $O(H (log(N) + B))$, where $B$ is a batch size. Otherwise can be slightly worse, up to $O(H log(N) B)$ in the worst case.
2. Point lookup (membership query) is $O(D H log(N))$.
3. Range and projection queries are $O(D H log(N) + M)$ _on average_, where $M$ is a recall size (number of vectors matching the query). In worst case tree traversal degrades to the linear scan of the entire set of vectors.
4. The data structure is space efficient. 2 bits per LOUDS tree node + _compressed bitmap_ of cardinal labels. For most usage scenarios, space complexity will be withing 2x the raw bit size of $str(\lbrace \vec{X} \rbrace)$.



