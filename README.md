# CFTP-using-a-SAT-solver

This repository contains an implementation of Coupling from the Past (CFTP) for sampling in various problems using a SAT solver.

## Problem Description

Given a very large but finite set $\Omega$ and a positive weight function $w : \Omega \to \mathbb{R}^+$, the goal is to sample $x \in \Omega$ with probability 

$$
\pi(x) = \frac{w(x)}{Z},
$$

where $Z = \sum_{x \in \Omega} w(x)$ is the normalizing factor, often called the partition function. Estimating $Z$ is a significant challenge in many applications.

Markov Chain Monte Carlo constructs a Markov Chain $M$ on $\Omega$ that converges to $\pi$ and produces a sample by simulating the Markov chain for sufficiently many steps.

### Coupling from the Past (CFTP)
CFTP is a Markov Chain sampling method with two highly desirable properties. The first is that it samples from the stationary distribution ***exactly***. The second is that it does not require ***any*** mathematical analysis to determine how many steps to take in order to draw a (perfect) sample. Instead, this is determined at run-time,  at the price of answering a co-NP question. 

CFTP works as follows: We produce random functions $f_{-1}, f_{-2}, \ldots$ from a distribution $\mathcal{F}$ on the set of all functions $f: \Omega \to \Omega$. We select $\mathcal{F}$ such that it is consistent with the Markov chain we are using; that is, 

$$ \Pr_{f \sim \mathcal{F}}[f(x) = y] = P(x, y), \ \forall x, y \in \Omega. $$

We call $\mathcal{F}$ a "random function" representation of $M$. Let

$$ F^j_i = f_{j-1} \circ f_{j-2} \circ \cdots \circ f_{i+1} \circ f_i. $$

The CFTP theorem guarantees that if $`T = \min\{t : F^0_{-t}\text{ is a constant function}\},`$
and $T$ is finite with probability 1, then the constant value $c = F^0_{-T}(x)$ has distribution exactly $\pi$.

In practice, we iteratively generate random functions $f_{-1}, f_{-2}, \ldots, f_{-t}$ until we can verify that $F^0_{-t}$ is constant. Since $\Omega$ is typically an exponentially large set, verifying this condition corresponds to a co-NP problem. To address this, we utilize a SAT solver, which checks whether $F^0_{-t}$ maps all elements of $\Omega$ to a single value.

## Sampling Problems

### Sampling Colorings

**Input:** A graph $G(V, E)$ and a positive integer $q$.

**Goal:** Produce a sample from the uniform distribution on the set of proper $q$-colorings of $G$.

**Markov Chain:** Glauber dynamics.

**Random Function Representation:**
- Select a node $v$ uniformly at random.
- Choose a uniformly random permutation of $[q]$.
- Recolor $v$ with the first color in $[q]$ that is available.

### Ferromagnetic Ising Model

**Input:** A graph $G(V, E)$ and a parameter $\lambda = e^{2\beta}$.

**Goal:** Produce a sample from the Gibbs distribution with parameter $\lambda$ on spin assignments $`\{-1, +1\}^{|V|}`$.

**Markov Chain:** Select a node $v$ uniformly at random.

Suppose $d^+$ neighbors of node $v$ have spin $+1$ and $d^-$ neighbors have spin $-1$, then set the spin of $v$ to $+1$ with probability:

$$
P^+ = \frac{\lambda^{d^+}}{\lambda^{d^+} + \lambda^{d^-}}
$$

and to $-1$ with probability

$$
P^- = \frac{\lambda^{d^-}}{\lambda^{d^+} + \lambda^{d^-}}.
$$


**Random Function Representation:** 
- Select a node $v$ uniformly at random.
- Choose $θ$ uniformly at random from the interval $[0,1]$.
- If $θ < P^-$, then the spin of node $i$ is set to $-1$, otherwise it is set to $+1$. This forms a monotone random function representation.

```

## References

1. Propp, James Gary, and David Bruce Wilson. "Exact sampling with coupled Markov chains and applications to statistical mechanics." Random Structures & Algorithms 9.1‐2 (1996): 223-252.
2. Dyer, Martin, et al. "Markov chain algorithms for sampling from the Ising model." SIAM Journal on Computing 25.5 (1996): 1046-1076.
