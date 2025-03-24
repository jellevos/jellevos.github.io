---
layout: page
title: Depth-aware arithmetization
description: Generating arithmetic circuits over finite fields that trade off multiplicative size and depth
img: assets/img/exponentiation.jpg
importance: 1
category: work
---

See: https://eprint.iacr.org/2024/1200

Homomorphic encryption schemes like BGV allow users to perform computations on data under encryption. Specifically, they can perform a number of additions and multiplications. There is a limit to the number of operations, induced by the noise that is added to these ciphertexts (for security purposes), and which grows after each addition or multiplication. We can increase the limit by choosing larger parameters, but these will slow down the computations.

The computation model described above is that of arithmetic circuits, and the process of turning a function into arithmetic is called <i>arithmetization</i>. If the plaintext space is a finite field (e.g. if the plaintext modulus $$p$$ is a prime), then any function can be turned into additions and multiplications. Doing so by hand, however, is a complex and time-consuming job. To prevent this, much research has been done to achieve efficient arithmetic circuits for primitives such as equality checks, comparisons, and OR operations. However, these circuits have typically not been generated with noise in mind.

The noise in an arithmetic circuit is mainly determined by its multiplicative depth: the largest number of multiplications in any path through the circuit. On the other hand, the computational cost is typically decided by the multiplicative size: the total number of multiplications in the circuit. However, the concrete cost of a multiplication is bound to the multiplicative depth. <b>Note that we only count multiplications between two variables; multiplications with constants are cheap to perform!</b>
In this post, we look at three methods for finding circuits that (near-)optimally trade off multiplicative size and depth.

<h2>Exponentiation & equality</h2>
Exponentiation circuits have been studied before for improving performance of elliptic curve multiplication, the RSA cryptosystem, and the AES cryptosystem. However, these works typically do not study the tradeoff between size and depth. One work that does consider this is that by <a href="https://arxiv.org/abs/2306.15002">Abbas & Gustafsson</a>, which uses integer linear programming to generate multiplication circuits with a maximum depth to compute a specific power. However, this approach is quite slow. We show how to reformulate this as a MaxSAT formulation and show that, by incorporating some additional bounds, we can compute the optimal Pareto front for powers lower than 100 in a few seconds.

A MaxSAT formulation is a SAT formulation with hard and soft clauses. Hard clauses must always be satisfied, but soft clauses may be dropped at the cost of a weight value. A MaxSAT solver then finds a solution to the clauses that incurs the lowest cost incurred by relaxing soft clauses. It does so by making repeated calls to an underlying SAT solver. Some solvers require the weight of soft clauses to be fixed, while others allow them to be any value. Varying the weights allows us to model that squaring is a cheaper operation than multiplication. In our experiments, we use the RC2 MaxSAT solver.

Our MaxSAT formulation computes some target exponent $$t$$. It uses variables $$x_i$$ for $$i = 1, \dots, t$$ to denote whether exponent $$i$$ has been reached in the circuit. Multiplications perform transitions between two inputs $$x_i$$ and $$x_j$$, and one output exponent $$x_{i+j}$$. We use variables $$y_{i,j}$$ to denote whether the circuit has performed a multiplication between $$x_i$$ and $$x_j$$. The hard clauses (presented in conjunctive normal form) are as follows:

$$
\bigvee_{i+j=k} y_{i,j} \vee \neg x_k,
$$

$$
\neg y_{i,j} \vee x_i,
$$

$$
\neg y_{i,j} \vee x_j,
$$

We consider $$i, j, k \in \{1, \dots, t\}$$. We also enforce $$x_t$$. These clauses ensure that a multiplication can only be performed when its inputs have previously been computed and that we must exponentiate to $$t$$. The soft clauses are simply $$y_{i,j}$$. We can use a variable weight to denote that $$y_{i,i}$$ is cheaper to compute than $$y_{i,j}$$ for $$i \ne j$$.

Abbas & Gustafsson already add a powerful bound to model that $$x_k$$ can only be true if the following holds: $$x_{\lceil k/2 \rceil} \vee \dots \vee x_{k-1}$$. We also add the bounds by <a href="https://www.sciencedirect.com/science/article/pii/S0012365X20303861">Thurber & Clift</a>, which provide a speedup for larger values of $$t$$. Next to that, since we compute the entire Pareto front, we can use knowledge of previous computations to compute a lower and upper bound on the cost of the formulation.

Now, to take depth into account, we use the same technique as Abbas & Gustafsson, but since we are working with Boolean variables we cannot perform addition as easily. We choose to model the depth of $$x_k$$ using several variables $$d_{k,d'}$$ for $$d' \in \{1, \dots, \text{max depth} \}$$. We change the formulation accordingly, by adding several hard clauses.

The resulting formulation can be used to compute optimal Pareto fronts for exponentiations in $$\mathbb{Z}$$. To take the modulus $$p$$ into account, we can sequentially compute optimal solutions for a target $$t + pi$$, until we are certain that no better circuit can be achieved. We use this technique to compute the optimal Pareto front for all exponents in $$\mathbb{Z}_p$$ in just 4 seconds on a laptop.

Why go through so much effort to find optimal exponentiation circuits? The answer is that they are a crucial building block of many common primitives, so any optimizations here have a large impact on the final performance. For example, to check whether an input $$z \in \mathbb{Z}_p$$ is non-zero, we can compute $$z^{p-1}$$. This fact can then immediately be used to implement an equality check $$a = b$$ as $$1 - (a - b)^{p-1}$$.

<h2>Univariate polynomial evaluation & comparisons</h2>
Arithmetic circuits compute multivariate polynomials. While it is easy to compute the multivariate polynomial that is computed by a circuit, the opposite is not true: it is extremely hard to find efficient circuits for arbitrary polynomials. Still, some common primitives do not have such natural formulations as the equality check shown above, so polynomial evaluation might be our best bet.

While multivariate polynomials are very hard to arithmetize efficiently, univariate polynomials are somewhat easier to study. Much work has gone into this area. One of the most fundamental results is by <a href="https://epubs.siam.org/doi/abs/10.1137/0202007?casa_token=jW3FzEnzE4MAAAAA:vHWKzedWp3uVmgJcU3dFkprLtSYpwuriISDVmLS9BQa962g3S33GEf0TkUbdMZ1P9wLEJFA7UtWH">Paterson and Stockmeyer</a>, who provide two algorithms for computing univariate polynomials of degree $$d$$. The first method is a baby-step giant-step approach, which takes roughly $$2\sqrt{d}$$ multiplications. The second method alters the first method to achieve roughly $$\sqrt{2d}$$ multiplications. They show that the multiplicative size of these circuits is asymptotically optimal.

These theoretical results are already of great practical value. However, Paterson & Stockmeyer only analyzed the multiplicative size of these circuits; not their depth. In fact, both methods have some parameter $$k$$ that must be set beforehand, and in choosing its value, the authors choose it to be optimal when it comes to the size of the circuit. We show that, by varying $$k$$, we can very efficiently produce a large amount of polynomial evaluation circuits of varying depth. While this does not produce provably optimal circuits, the circuits are close to optimal for small values of $$d$$.

In previous work, <a href="https://hal.science/hal-03506798/">Iliashenko & Zucca</a> show how to efficiently express comparisons such as $$a < b$$ as a univariate polynomial, specifically when $$a, b < \frac{p-1}{2}$$. They use properties from the second method by Paterson & Stockmeyer to reduce the multiplicative size of the circuit. To amend the first method of Paterson & Stockmeyer to this same trick, we use the MaxSAT formulation above. Finally, we show that one can lift the restriction that $$a, b < \frac{p-1}{2}$$ by computing three polynomial evaluations.

<h2>AND and OR operations over multiple variables</h2>
In the above two examples (equality checks and comparisons), the circuit does not change if one of the inputs has a lower depth than the other. This is because the first step in computing them requires subtracting them. The multiplicative depth after subtraction is the maximum of the two. AND and OR circuits, on the other hand, change significantly depending on the depths of the inputs. We assume that the inputs are in $$\{ 0, 1\}$$.

The most common way to arithmetize an AND operation is to use a simple product:
$$x_1 \wedge x_2 \wedge \dots \wedge x_k = x_1 x_2 \dots x_k$$

The result is a circuit of depth $$\lceil \log_2 \sum_{i=1}^k \operatorname{depth}(x_i) \rceil$$. If the variables are completely independent (i.e. there are no common subexpressions), then the size of the resulting circuit is $$k-1 + \sum_{i=1}^k \operatorname{size}(x_i)$$. This is not the only efficient arithmetization, however. Consider the following arithmetization of an OR operation with $k \le p-1$:

$$x_1 \vee x_2 \vee \dots \vee x_k = (x_1 + x_2 + \dots + x_k)^{p-1}$$

This circuit adds up the inputs so they do not overflow the modulus, and then checks if the result is non-zero. This circuit's depth is $$D + \max_{i=1,\dots,k} \operatorname{depth}(x_i)$$, where $$D$$ is the depth of the exponentiation circuit (see first section). The size is $$S + \sum_{i=1}^k \operatorname{size}(x_i)$$, where $$S$$ is the size of the exponentiation circuit.

Arithmetizations for AND operations can easily be translated to OR operations (and the other way around) using DeMorgan's law:
$$x_1 \vee x_2 \vee \dots \vee x_k = \overline{\overline{x_1} \wedge \overline{x_2} \wedge \dots \wedge \overline{x_k}}$$

These arithmetization can be combined into hybrid circuits that find a tradeoff between depth and size. The smallest practical example of this happens when $$k = 7$$ and $$p = 5$$. The circuit generated using the first arithmetization has size 6 and depth 3 (circuit A). However, when we consider the hybrid solution, we can generate two better circuits. The first has depth 4 and size 4 (circuit B), the second has depth 3 and size 5 (circuit C).

Given these two arithmetic circuits, we do not know which one is more performant in practice. We use HElib and fhegen to gather some preliminary results. For fhegen we used the OpenFHE model as it does not include rotations (we do not consider ciphertext packing). The results, when computed on an M1 CPU, are as follows: Circuit A takes 36.2 ms, circuit B takes 41.3 ms, circuit C takes 22.8 ms. Circuits A and C had a lower depth, which allowed the use of a smaller ring dimension. Moreover, circuit C had a lower cost than circuit A as it has fewer multiplications (and two multiplications are squarings).

~We are currently integrating these algorithms in the <a href="https://github.com/jellevos/oraqle">oraqle compiler</a> for secure computation, which we plan to release in April.~
The oraqle compiler is available! Some of the early features are described in: https://eprint.iacr.org/2024/1409

