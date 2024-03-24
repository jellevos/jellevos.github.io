---
layout: page
title: Depth-aware arithmetization
description: Generating arithmetic circuits over finite fields that trade off multiplicative size and depth
img: assets/img/exponentiation.jpg
importance: 1
category: work
---

<i>An eprint of this work will be out soon! I will update this page when it is ready.</i>

Homomorphic encryption schemes like BGV allow users to perform computations on data under encryption. Specifically, they can perform a number of additions and multiplications. There is a limit to the number of operations, induced by the noise that is added to these ciphertexts (for security purposes), and which grows after each addition or multiplication. We can increase the limit by choosing larger parameters, but these will slow down the computations.

The computation model described above is that of arithmetic circuits, and the process of turning a function into arithmetic is called <i>arithmetization</i>. If the plaintext space is a finite field (e.g. if the plaintext modulus is a prime), then any function can be turned into additions and multiplications. Doing so by hand, however, is a complex and time-consuming job. To prevent this, much research has been done to achieve efficient arithmetic circuits for primitives such as equality checks, comparisons, and OR operations. However, these circuits have typically not been generated with noise in mind.

The noise in an arithmetic circuit is mainly determined by its multiplicative depth: the largest number of multiplications in any path through the circuit. On the other hand, the computational cost is typically decided by the multiplicative size: the total number of multiplications in the circuit.
In this post, we look at three methods for finding circuits that (near-)optimally trade off multiplicative size and depth.

<h2>Exponentiation & equality</h2>

<h2>Univariate polynomial evaluation & comparisons</h2>

<h2>AND and OR operations over multiple variables</h2>
