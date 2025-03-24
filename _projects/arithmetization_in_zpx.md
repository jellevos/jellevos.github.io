---
layout: page
title: Arithmetization in the polynomial ring
description: Generating arithmetic circuits that exploit batching and automorphisms
importance: 1
category: work
---

[In a previous blog post](/projects/depth_aware_arithmetization), we presented methods for performing depth aware arithmetization in prime fields $\mathbb{F}_p$. In other words, we showed how to realize high-level circuits in $\mathbb{F}_p$, which only defines additions and multiplications. What made these methods <i>depth-aware</i>, is that we generated circuits that achieve a trade off between their multiplicative depth (the largest number of multiplications on any path through the circuit) and the total number of multiplications. These methods allowed us to generate more efficient circuits for e.g. comparisons ($a < b$) than previous works. However, these methods leave several things to be desired: they do not consider batching and automorphisms. In this blog post, we will show how to consider automorphisms by extending beyond prime fields, and how to further optimize circuits in the context of homomorphic encryption by exploiting batching in the polynomial ring.

<h2>Arithmetization in $\mathbb{F}_{p^d}$: Exploiting automorphisms</h2>
In the finite fields $\mathbb{F}_p$ that we previously considered, all the useful operations are described by additions and multiplications. In those prime fields, the only automorphism is the identity map. However, when we consider non-prime fields $\mathbb{F}_{p^e}$, there are several useful automorphisms of the form $x \mapsto x^{p^k}$, where $0 < k < e$.

<h2>Arithmetization in $\mathbb{Z}_p/(X^N + 1)$: Exploiting batching</h2>
Note: we only consider $N$ that are powers of two!


<h2>Polynomial evaluation in $\mathbb{Z}_p/(X^N + 1)$</h2>
