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

The computation model described above is that of arithmetic circuits, and the process of turning a function into arithmetic is called <i>arithmetization</i>. If the plaintext space is a finite field (e.g. if the plaintext modulus $$p$$ is a prime), then any function can be turned into additions and multiplications. Doing so by hand, however, is a complex and time-consuming job. To prevent this, much research has been done to achieve efficient arithmetic circuits for primitives such as equality checks, comparisons, and OR operations. However, these circuits have typically not been generated with noise in mind.

The noise in an arithmetic circuit is mainly determined by its multiplicative depth: the largest number of multiplications in any path through the circuit. On the other hand, the computational cost is typically decided by the multiplicative size: the total number of multiplications in the circuit. However, the concrete cost of a multiplication is bound to the multiplicative depth.
In this post, we look at three methods for finding circuits that (near-)optimally trade off multiplicative size and depth.

<h2>Exponentiation & equality</h2>
Exponentiation circuits have been studied before for improving performance of elliptic curve multiplication, the RSA cryptosystem, and the AES cryptosystem. However, these works typically do not study the tradeoff between size and depth. One work that does consider this is that by <a href=https://arxiv.org/abs/2306.15002>Abbas & Gustafsson</a>, which uses integer linear programming to generate multiplication circuits with a maximum depth to compute a specific power. However, this approach is quite slow. We show how to reformulate this as a MaxSAT formulation and show that, by incorporating some additional bounds, we can compute the optimal Pareto front for powers lower than 100 in a few seconds.

A MaxSAT formulation is a SAT formulation with hard and soft clauses. Hard clauses must always be satisfied, but soft clauses may be dropped at the cost of a weight value. A MaxSAT solver then finds a solution to the clauses that incurs the lowest cost incurred by relaxing soft clauses. It does so by making repeated calls to an underlying SAT solver. Some solvers require the weight of soft clauses to be fixed, while others allow them to be any value. Varying the weights allows us to model that squaring is a cheaper operation than multiplication. In our experiments, we use the RC2 MaxSAT solver.

Our MaxSAT formulation computes some target exponent $$t$$. It uses variables $$x_i$$ for $$i = 1, \dots, t$$ to denote whether exponent $$i$$ has been reached in the circuit. Multiplications perform transitions between two inputs $$x_i$$ and $$x_j$$, and one output exponent $$x_{i+j}$$. We use variables $$y_{i,j}$$ to denote whether the circuit has performed a multiplication between $$x_i$$ and $$x_j$$. The hard clauses (presented in conjunctive normal form) are as follows:

$$
\bigvee_{i+j=k} y_{i,j} \vee \neg x_k \\
\neg y_{i,j} \vee x_i \\
\neg y_{i,j} \vee x_j \\
x_t
$$

We consider $$i, j, k \in \{1, \dots, t\}$$. These clauses ensure that a multiplication can only be performed when its inputs have previously been computed and that we must exponentiate to $$t$$. The soft clauses are simply $$y_{i,j}$$. We can use a variable weight to denote that $$y_{i,i}$$ is cheaper to compute than $$y_{i,j}$$ for $$i \ne j$$.

Abbas & Gustafsson already add a powerful bound to model that $$x_k$$ can only be true if the following holds: $$x_{\lceil k/2 \rceil} \vee \dots \vee x_{k-1}$$. We also add the bounds by <a href=https://www.sciencedirect.com/science/article/pii/S0012365X20303861>Thurber & Clift</a>, which provide a speedup for larger values of $$t$$. Next to that, since we compute the entire Pareto front, we can use knowledge of previous computations to compute a lower and upper bound on the cost of the formulation.

Now, to take depth into account, we use the same technique as Abbas & Gustafsson, but since we are working with Boolean variables we cannot perform addition as easily. We choose to model the depth of $$x_k$$ using several variables $$d_{k,d'}$$ for $$d' \in \{1, \dots, \text{max depth} \}. We change the formulation accordingly, by adding several hard clauses.

The resulting formulation can be used to compute optimal Pareto fronts for exponentiations in $$\mathbb{Z}$$. To take the modulus $$p$$ into account, we can sequentially compute optimal solutions for a target $$t + pi$$, until we are certain that no better circuit can be achieved. We use this technique to compute the optimal Pareto front for all exponents in $$\mathbb{Z}_p$$ in just 4 seconds on a laptop.

Why go through so much effort to find optimal exponentiation circuits? The answer is that they are a crucial building block of many common primitives, so any optimizations here have a large impact on the final performance. For example, to check whether an input $$z \in \mathbb{Z}_p$$ is non-zero, we can compute $$z^{p-1}$$. This fact can then immediately be used to implement an equality check $$a = b$$ as $$1 - (a - b)^{p-1}$$.

<h2>Univariate polynomial evaluation & comparisons</h2>
Method and comparisons

<h2>AND and OR operations over multiple variables</h2>
Some dots on the front

<h2>Composition</h2>
