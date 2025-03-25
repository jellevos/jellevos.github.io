---
layout: page
title: Arithmetization in the polynomial ring
description: Generating arithmetic circuits that exploit batching and automorphisms
importance: 1
category: work
---

[In a previous blog post](/projects/depth_aware_arithmetization), we presented methods for performing depth aware arithmetization in prime fields \\(\mathbb{F}_p\\). In other words, we showed how to realize high-level circuits in \\(\mathbb{F}_p\\), which only defines additions and multiplications. What made these methods <i>depth-aware</i>, is that we generated circuits that achieve a trade off between their multiplicative depth (the largest number of multiplications on any path through the circuit) and the total number of multiplications. These methods allowed us to generate more efficient circuits for e.g. comparisons (\\(a < b\\)) than previous works. However, these methods leave several things to be desired: they do not consider batching and automorphisms. In this blog post, we will show how to consider automorphisms by extending beyond prime fields, and how to further optimize circuits in the context of homomorphic encryption by exploiting batching in the polynomial ring.

<h2 data-processed="0">Arithmetization in \(\mathbb{F}_{p^d}\): Exploiting automorphisms</h2>
In the finite fields \(\mathbb{F}_p\) that we previously considered, all the useful operations are described by additions and multiplications. In those prime fields, the only automorphism is the identity map. However, when we consider non-prime fields \\(\mathbb{F}_{p^d}\\), there are several useful automorphisms of the form \\(x \mapsto x^{p^k}\\), where \\(0 < k < d\\). When \\(k=1\\), we call this the Frobenius automorphism. As such, where arithmetic circuits over \\(\mathbb{F}_p\\) were only comprised of additions and multiplications, we now consider additions, multiplications, and automorphisms.

While these automorphisms may not seem all that useful by themselves, they can be used to efficiently compute the field norm and trace. These operations, in turn, can be used to (more) efficiently evaluate polynomials, as shown by Hiroki Okada, Rachel Player, and Simon Pohmann in [this article](https://eprint.iacr.org/2023/1304), or to speed up matrix multiplications, as shown by Jai Hyun Park in [this other article](https://eprint.iacr.org/2025/448). In this blog post, we will focus on univariate polynomial evaluation, which allows us to compute comparison operations even more efficiently than before!

So how does this work? The trick is that there is a relation between the minimal polynomial of an element \\(\alpha \in \mathbb{F}_{p^d}\\) and its field norm:
$$ \operatorname{Norm}(\alpha - x) = \operatorname{MiPo}(\alpha)(x), $$
where \\(\operatorname{MiPo}(\alpha)\\) is the monic (univariate) polynomial of the lowest degree that maps \\(\alpha\\) to zero.
Because no lower degree polynomials exist that also map \\(\alpha\\) to zero, the minimal polynomial is irreducible.
This relation allows us to evaluate any irreducible polynomial over \\(\mathbb{F}_p\\) by finding a suitable \\(\alpha\\) and computing the field norm of \\(\alpha - x\\).

So how do we find such a suitable \\(\alpha \in \mathbb{F}_{p^d}\\)? 
To evaluate an irreducible monic polynomial with coefficients in \\( \mathbb{F}_p \\), we want to find its root in the polynomial ring \\(\mathbb{F}_{p^d}[X]\\).
Our current approach is to factor the polynomial in this polynomial ring.
For larger polynomials, this can be a costly process, but it only has to be computed once (\\(\alpha \\) does not depend on the inputs of the polynomial evaluation).

The reason that this approach leads to more efficient evaluations, is that whereas previous methods required \\( O(\sqrt{D}) \\) multiplications to compute a degree-\\(D\\) polynomial, the norm can be computed in only \\( 2 \log_2(D) \\) multiplications and automorphisms. As such, it leads to significantly more efficient circuits when evaluating large polynomials. 

How can we evaluate reducible polynomials?
Just like Hiroki Okada, Rachel Player, and Simon Pohmann, we add an easily computable term to the polynomial that makes it irreducible, and we remove this term after the polynomial evaluation. In our experiments, it has always been sufficient to add a simple constant to the polynomial.

<h2 data-processed="0">Arithmetization in \( \mathbb{Z}_p/(X^N + 1) \): Exploiting batching</h2>
<i>Note: we only consider \\(N\\) that are powers of two!</i>

Batching (or the SIMD encoding) is a ubiquitous tool in fully homomorphic encryption schemes such as BFV and BGV because it allows one to pack multiple elements into one ciphertext and perform the same additions and multiplications on all of them. The oraqle compiler so far has not considered batching at all, so the only way to exploit batching was to evaluate the circuit many times in parallel. However, batching can also be used when we only want to compute one evaluation of an arithmetic circuit.

SPLIT INTO FPd
GRID

FUNCTIONAL PROGRAMMING

MAPS, REDUCES, SPLICING.

CONSIDER HOW MANY ELEMENTS TO PACK.


<h2 data-processed="0">Polynomial evaluation in $\mathbb{Z}_p/(X^N + 1)$</h2>
In our [article on depth-aware arithmetization](https://eprint.iacr.org/2024/1200), we proposed a simple divide-and-conquer technique for evaluating large-degree polynomials that often produces lower-depth solutions than previous work such as the baby-step giant-step method and the Paterson-Stockmeyer method. We can also use this technique to evaluate polynomials of larger degrees by splitting it up into multiple smaller univariate polynomial evaluations. This, in turn, allows us to use the efficient automorphism-based approach described above. We take the following steps:

- We split the large-degree polynomial up into evaluations of degree-\\(D\\) polynomials
- We use the Map node to evaluate all degree-\\(D\\) polynomials in parallel using automorphisms
- We use the Reduce node to combine the results, multiplying the appropriate terms with powers of \\(x\\)

By using the functional programming nodes defined above, the compiler can find the best way to pack these evaluations into multiple ciphertexts. 

We are still implementing these techniques in the [oraqle compiler compiler](https://github.com/jellevos/oraqle), and we expect to take a month or two to release the next version. Behind the scenes, there a lot of changes! For example, we now require supporting multiple plaintext algebras and the functional programming nodes add a completely new optimization step. We are also still experimenting with some heuristics for merging multiple functional programming nodes automatically.
