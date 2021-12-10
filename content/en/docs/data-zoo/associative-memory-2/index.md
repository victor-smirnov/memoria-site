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

Let $\vec{X}$ is a vector encoding a point in $[0, 1]^D$, $\vec{X} = <x_0, x_1, ..., x_{D-1}>$. Let $str(x [, h])$ is a function converting a real number from $[0, 1]$ into a binary string by taking a binary representation of a real number to a form like "$0.0010111010...$", and removing leading "$0.$" and trailing "$0...$". so, $str(0.181640625) = 001011101$. If $h$ argument is specified for $str(\cdot, \cdot)$, then resulting string is trimmed to $h$ binary digits, if it's longer than that.

Let $len(x)$ is a number of digits in the result of $str(x)$. Note that $len(\pi / 10) = \infty$, so irrational numbers are literally infinite in this notation. Let $H$ is a maximal _depth_ of data, and there is some _implicitly assumed_ arbitrary value for $h$, like 32 or 64, or even 128. So we can work with _approximations_ of irrational and transcendent numbers, or with long rational numbers in a same way and without loss of generality.

Let $len(\vec{X}) = max_{\substack{i \in \lbrace 0,...,D-1 \rbrace }}(len(x_i))$.

Let $str(\vec{X}) = (str(x_0),..., str(x_{D-1}))$ is a string representation (a tuple) of $\vec{X}$. And let we assume, elements of the tuple are implicitly extended with '$0$' from the right, if their length is less than $len(\vec{X})$. In other words, all elements (binary string) of a tuple are implicitly of the same length.

Let $str(\lbrace \vec{X_0}, \vec{X_1}, ... \rbrace) = \lbrace str(\vec{X_0}), str(\vec{X_1}), ... \rbrace$. String representation of set of vectors is a set of string representation of individual vectors.

Let $M(\vec{X}) = M(str(\vec{X}))$ is a [multiscale transformation](/docs/data-zoo/associative-memory-1/#multiscale-decomposition) of binary string representation of $\vec{X}$. Informally, to compute $M(\vec{X})$ we need to take all strings from its string representation (the tuple of binary strings) and produce another string of length $len(\vec{X})  D$ by concatenating interleaved bits from each binary string in the tuple.

Example. Let $H = 3$ and $D = 2$. $\vec{X} = <0.625, 0.25>$, $str(\vec{X}) = (101, 01)$ and $M(\vec{X}) = 10|01|10$. Note that signs $|$ are added to separate $H$ _path elements_ in the recording, they are here for the sake of visualization and are not a part of the representation. The string $10|01|10$ is also called _path expression_ because it's a unique path in the multidimensional space decomposition _encoding_ the position of $\vec{X}$ in this decomposition. Visually:

 {{< figure src="multiscale1.svg" width="100%" >}}
 
 Note that the order in which _path elements_ of a path expression enumerate the volume is [Z-order](https://en.wikipedia.org/wiki/Z-order_curve). 
 
Given a _set_ of $D$-dimensional vectors $\lbrace \vec{X} \rbrace$, it's multiscale transformation can be represented as a $D$-dimensional [Quad Tree](https://en.wikipedia.org/wiki/Quadtree). Such quad tree can be represented with a [cardinal LOUDS tree](/docs/data-zoo/louds-tree/#cardinal-trees) of degree $2^D$. Here, implicit parameter $H$ is a _maximal height_ of the Quad Tree. 
 
## Basic Asymptotic Complexity 
 
Given that $N = |\lbrace \vec{X} \rbrace|$, and given that LOUDS tree is dynamic (represented internally as a b-tree), the following complexity estimations apply:

1. **Insertion and deletion** of a point is $O(H log(N))$. Update is semantically not defined because it's a _set_. Batch updates in Z-order are $O(H (log(N) + B))$, where $B$ is a batch size. Otherwise can be slightly worse, up to $O(H log(N) B)$ in the worst case.
2. **Point lookup** (membership query) is $O(D H log(N))$.
3. **Range and projection** queries are $O(D H log(N) + M)$ _on average_, where $M$ is a recall size (number of vectors matching the query). In worst case tree traversal degrades to the linear scan of the entire set of vectors.
4. The data structure is **space efficient**. 2 bits per LOUDS tree node + _compressed bitmap_ of cardinal labels. For most usage scenarios, space complexity will be within **2x the raw bit size** of $str(\lbrace \vec{X} \rbrace)$.


## Functions Approximation Using Quad Trees

Let we have some function $y = f(x)$, and we also have the graph of this function on $[0, 1]^2$. If function $f(\cdot)$ is elementary, or we have another way to compute it, it's computable (for us). What if we have $f(\cdot)$, but we want to compute inverse function: $x = f^{-1}(y)$? With compact quad trees we can _approximate_ both functions out of the same _function graph_ using compact quad trees:

{{< figure src="function.svg" width="100%"  >}}

Here the tree has four layers shown upside down (drawing more detailed layers above less detailed ones). And if we want to compute $f(a)$, where $str(a) = a_3a_2a_1a_0$, the path expression will be $a_3x|a_2x|a_1x|a_0x$, where $x$ here is a _placeholder sign_. Now we need to traverse the tree as it defined in [previous part](/docs/data-zoo/associative-memory-1/#multiscale-decomposition). If there is a point for $a$ of a function graph, it will be found in a logarithmic expected time. The path expression for $f^{-1}(b)$ will be $b_3x|b_2x|b_1x|b_0x$.

Note that compressed cardinal LOUDS tree will use less than 4 bits (+ some % of auxiliary data) _per square_ on the graph above. Sol, looking into this graph we already can say something specific about what will be the cost of approximation, depending on required precision (maximal $H$).

## Function Compression

Let we have a function that checks if some point is inside some ellipse: 

{{<m>}}
y = f(x_1, x_2): \begin{cases} 
    1 &\text{if } (x_1, x_2) \text{ is inside the the ellipse,} \\ 
    0 &\text{if it's outside.} 
\end{cases}
{{</m>}}



This function defines some _area_ on the graph. Let $N$ is the number of "pixels" we need to define the function $f(x,y)$ on a graph. Then, using compressed quad trees, we can do it with ${O(\sqrt{N})}$ bits _on average_ for 2D space :

{{< figure src="region.svg" >}}

The square root here is because we need "detailed" only for the border of the area, while "in-lands" needs much lower resolution. 

## Blessing of Dimensionality vs Curse of Dimensionality

Let we have 2D space and we have a tree encoding the following structure:

{{< figure src="corners.svg" >}}

There are four points in each corner of the coordinate plane. As it can be obvious from the picture, each new level of the tree will add four new bits for just for cardinal labels (not including LOUDS tree itself): three '0' and one '1' (in the corresponding corners). Now if we go to higher dimensions, for 3D we will have 8 new bits, for 8D -- 256 new bits and for 32D -- $2^32$ bits. This is **Curse of Dimensionality** (CoD) for spatial data structures: volume grows exponentially with dimensions, so linear sizes -- too.

Nevertheless, with higher dimensions, the volume is getting exponentially sparser, so we can use data compression techniques like RLE encoding to represent long sparse bitmaps for cardinal labels. This is **Blessing of Dimensionality** (BoD). For _compressed_ LOUDS cardinal trees, the example above will require $O(D)$ bits per quadrant per tree layer for $D$-dimensional Quad Tree.

So, the the whole idea of compression in this context is implicit (or in-place) **Dimensionality Reduction**. Compressed data structure don't degrade so fast as their uncompressed analogs, yet maintain the same _logical API_. Nevertheless, data compression is not the final cure for CoD, because practical compression itself is not that powerful, especially in the case of using RLE for bitmap compression. So, in each practical case high-dimensional tree can become "unstable" and "explode" in size. Fortunately, such highly-dimensional data ($D > 16$) is rarely makes sense to work with directly (without prior dimensionality reduction). 

For example, for $D=8$ exponential effects in space are still pretty moderate (256-degree cardinal tree), yet 8 dimensions is already a good approximation of real objects in symbolic methods. High-dimensional structures are effectively _black boxes_ for us, because our visual intuitions about properties of objects don't work in [high dimensions](https://www.math.wustl.edu/~feres/highdim). Like, volume of cube is concentrating in it's corners (because there is an exponentional number of corners). Or the volume of sphere is concentrating near its surface, and many more... Making decisions in high dimensions suffer from noise in data and machine rounding, because points tend to be very close to each other. And, of course, computing Euclidian distance does not make (much) sense. 

## Comparison with Multi-Layer Perceptrons

Neural networks has been known to be a pretty good function approximators, especially for multi-dimensional cases. Let's check how compressed spatial tree can be compared with multi-layer perceptrons (MLP). This type of artificial neural networks is by no means the best example of ANNs, yet it's a pretty ideomatic member of this family. 

In the core of MLP is the idea of _linear separability_. In a bacis case, there are two regions of multidimensional space that can't be separated by a hyperplane from each other. MLP has multiple ($K$) layers, where first $K-1$ layers perform specific space transformations using linear (weights) and non-linear (thresholds) operators in such way that $K$-th layer can perform the linear separation: 

{{< youtube k-Ann9GIbP4 >}}

So, this condition is simplified of the picture below. Here, we have two classes (_red_ and _green_ dots) with complex non-linear boundary between those classes. After transforming the space towards linear separation of those classes and making inverse transformation, the initial hyperplane (here, a line) is broken in many places:

{{< figure src="classifier.svg" >}}

What is important, is that each such like break is an inverse transformation of the original hyperplane. Those transformations has to be described, and description will take some space. So we can speak about the **descriptional (Kolmogorov) complexity of the decision boundary**. Or, in the other way, _how many neurons (parameters) we need to encode the decision boundary_?

From Algorithmic Information Theory it's known that arbitrary string $s$, drawn from a uniform distribution, will be _incompressible_ with high probability, or $K(s) \to |s|$. In other words, most mathematically possible objects are _random_-looking, we hardly can find and exploit any structure in them. 

Returning back to MLP, it's expected that in "generic case" decision boundaries will be _complex_: the line between classes will have many breaks, so, may transformations will be needed to describe it with required precision (and this is even not taking CoD into account).

Describing decision boundaries (DB) with compressed spatial trees may look like a bad idea from the first glance. MLPs encode DB with superpositions of elementary functions (hyperplanes and non-linear units). Quad Trees do it with hyper-cubes, and it's obvious that we may need a lot of hyper-cubes in place of just one arbitrary hyper-plane. If it's the case, we say that hyper-planes _generalize_ DB _much better_ than hyper-cubes: 

{{< figure src="classifier-tree.svg" >}}

But it should hardly be a surprise, if in a real life case it will be find out that descriptions or the network and the corresponding tree are roughly the same.

So, Neural Networks may generalize much better in some cases than compressed quad trees and perform better in very high dimensional spaces (they suffer less from CoD because of better generalization), but trees have the following benefits:

1. If computational complexity of MLP is $\Omega(W)$ (a lot of large matrix multiplications), where $W$ is number of parameters, complexity of inference in the quad tree is _roughly_ from $O(log(N))$, where $N$ is number of bits of information in the tree. So trees may be much faster than networks _on average_. 
1. Quad Trees are dynamic. Neural Networks require retraining in case of updates, at the same time adding (or removing) an element to the tree is $O(log(N))$ _worst case_. It may be vital for may applications operating on-line, like robotics.
1. Quad Trees support "inverse inference" mode, when we can specify classes (outputs) and see 

So, compressed quad trees may be much better for complex dynamic domains with tight decision boundaries with moderate number of dimensions (8-24). It's not that clear yet how trees will perform in real life applications. Memoria is providing (1) _experimental_ compressed dynamic cardinal LOUDS tree for low dimensional spaces (2 - 64).

(1) Not yet ready at the time of writing.

## Hardware Acceleration

The main point of hardware acceleration of compressed Quad Trees is that inference may be really cheap (on average). It's just a bunch of memory lookups (it's a _memory-bound_ problem). Matrix multipliers, from other size, are also pretty energy efficient. Nevertheless, _trees scale better with complexity of decision boundaries_.
