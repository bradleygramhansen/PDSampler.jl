# About PDMP samplers

This page aims at giving a very brief introduction to the concept of PDMP samplers (below we will refer to *the algorithm* but it should be understood as a class of algorithms). We also give some insight into how it is implemented although we cover the implementation in more details in the technical documentation. This is not meant to be a rigorous presentation of the algorithm (for this, please see the references at the bottom of this page). Rather, we focus here on the "large building blocks" behind the algorithm.

## Basic idea (global samplers)

The purpose of the algorithm is to be able to evaluate expected values with respect to an arbitrary target distribution which we assume admits a probability density function  $\pi$. For simplicity, we assume that $\pi:C\to \mathbb R^+$ with $C\subseteq \mathbb R^p$, convex. The objective is therefore to compute a weighted integral of the form:

\begin{equation}
    \mathbb E_{\pi}[\varphi(X)] = \int_{C} \varphi(x)\pi(x)\,\mathrm{d}x
\end{equation}

The samples considered generate a *piecewise-linear path*

\begin{equation}
    x(t) = x^{(i)} + v^{(i)}(t-t_i) \quad \text{for}\quad t\in[t_i, t_{i+1}]
\end{equation}

determined by an initial position $x^{(0)}$ and velocity $v^{(0)}$ at time $t_0=0$ and a set of positive *event times* $t_1,t_2,\dots$. Under some conditions for the generation of the times and the velocities, the expected value can be approximated with

\begin{eqnarray}
    \mathbb E_{\pi}[\varphi(X)] &\approx& {1\over T} \int_0^T\varphi(x(t))\mathrm{d}t
\end{eqnarray}

and the integral in the right hand side can be expressed as a sum of one-dimensional integrals along each linear segment.

### Generating times and velocities

The algorithm generates a sequence of *triples* of the form $(t^{(i)}, x^{(i)}, v^{(i)})$.
Let us assume that the algorithm is currently at one of those event points and show how to compute the next triple. To do so, the algorithm executes the following steps:

1. it generates a travel time $\tau$ drawing from a specific process,
2. the next position is obtained by traveling along the current *ray* for the travel time $\tau$ i.e.: $x^{(i+1)} = x^{(i)} + \tau v^{(i)}$,
3. a new velocity $v^{(i+1)}$ is generated.

First we will explore how the travel time is generated and then how the new velocity is computed.

#### Sampling a travel time

The travel time $\tau$ is obtained as the minimum of three times which we will denote by $\tau_b, \tau_h, \tau_r$. Following the case, the computation of the new velocity will be different.

The first (and most important) one, $\tau_b$, is the first arrival time of an *Inhomogenous Poisson Process* (IPP) with an intensity that should verify some properties with respect to the target distribution. The *Bouncy Particle Sampler* (BPS) in particular considers the following intensity with $U$ the log-likelihood of the (unnormalised) target $\pi$:

\begin{eqnarray}
    \lambda(\tau; x, v) = \langle \nabla U(x + \tau v ), v \rangle^+
\end{eqnarray}

where $x$ and $v$ are the current points and $f^+=\max(f,0)$. Sampling from an IPP is not trivial in general but there are a few well known techniques that can be applied depending on the target.

#### Computing a new velocity

As mentioned above, we take $\tau = \min(\tau_b, \tau_h, \tau_r)$

1. a **bounce** with $\tau = \tau_b$ where a velocity is recomputed following the value of the gradient of the log-likelihood of the target (see below),
2. a **boundary bounce** with $\tau=\tau_{h}$ where the velocity is reflected against a boundary of the domain $C$,
3. a **refreshment** with $\tau=\tau_r$ where the velocity is "refreshed".

A time $\tau$ is drawn, if  $\tau\le \min(\tau_h,\tau_r)$, step (1) is applied. If $\tau \ge \tau_r$ step (3) is applied. Otherwise step (2).
Both $\tau_h$ and $\tau_r$ should be considered given. The first one, $\tau_h$ is the hitting time between the ray and the closest boundary of $C$ (for simple domains like a polygonal domain it can be computed analytically). The second one, $\tau_r$ is drawn from an exponential distribution (this allows to guarantee that the algorithm explores the whole space).

It remains to explain how to generate $\tau$ and how the velocity is updated.



The update of the velocity goes as follows for the BPS:

* (**bounce**) the new velocity is obtained by computing a specular reflection of the velocity against the tangent to the gradient of the log-likelihood at $x(t+\tau_b)$ is computed

\begin{equation}
    v \leftarrow v - 2\langle \nabla U(x), v\rangle{\nabla U(x)\over \|\nabla U(x)\|^2}
\end{equation}

* (**boundary hit**) the new velocity is obtained by computing a specular reflection against the tangent to the boundary at the hitting point.
* (**refresh**) the new velocity is obtained by sampling from a "refreshment distribution" for example a $\mathcal N(0, I)$.

The illustration below illustrates the specular reflexion, starting at the red point and going along the ray (red, dashed line), we could have a new event corresponding to bounce or a hit (blue dot). In both case a specular reflection is executed (blue dashed line).

![](assets/BPS.svg)


### Sampling from an IPP



## Local Samplers

## References

* Alexandre Bouchard-Côté, Sebastian J. Vollmer and Arnaud Doucet, [*The Bouncy Particle Sampler: A Non-Reversible Rejection-Free Markov Chain Monte Carlo Method*](https://arxiv.org/abs/1510.02451), arXiv preprint, 2015.
* Joris Bierkens, Alexandre Bouchard-Côté, Arnaud Doucet, Andrew B. Duncan, Paul Fearnhead, Gareth Roberts and Sebastian J. Vollmer, [*Piecewise Deterministic Markov Processes for Scalable Monte Carlo on Restricted Domains*](https://arxiv.org/pdf/1701.04244.pdf), arXiv preprint, 2017.
* Joris Bierkens, Paul Fearnhead and Gareth Roberts, [*The Zig-Zag Process and Super-Efficient Sampling for Bayesian Analysis of Big Data*](https://arxiv.org/pdf/1607.03188.pdf), arXiv preprint, 2016.