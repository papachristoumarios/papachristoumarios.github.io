---
layout: post
title: "Sampling from high-dimensional (truncated) log-concave densities with GeomScale: A Gentle Introduction"
date: 2020-07-21 17:00:00 +0200
categories:
---

> This article serves as a reference article for my _Google Summer of Code 2020_ internship at the GeomScale organization. The project repository can be found [here](https://github.com/GeomScale/volume_approximation).

**THE ARTICLE IS A WORK IN PROGRESS**



## Problem Definition

### Introduction

A particularly interesting problem in statistics is taking samples from a density of the form $$\pi(x) \propto \exp(-f(x))$$ where $$f(x)$$ is a convex function. To give the interested reader a quick glance into the problem, consider the _logistic regression_ problem where $$f$$ has a form of

<center>
$$f(\theta) = \frac 1 m \sum_{i = 1}^m \log(\exp(-y_i x_i^T \theta) + 1)$$
</center>
where $$\{x_i, y_i \}_{i =1}^m$$ is a set of $$m$$ samples. This kind of distribution is called _log-concave_ since the logarithm of $$\pi(x) \propto \exp(-f(x))$$ is a _concave function_. _Log-concave_ distributions are [pretty common](https://en.wikipedia.org/wiki/Logarithmically_concave_function#Log-concave_distributions), and emerge in a variety of fields: classical statistics, statistical physics, financial economics, neural networks, and optimal control; to name a few. Getting samples from a log-concave density directly poses a significant problem since one has to calculate the normalization constant 

<center>
	$$Z = \int_K \exp(-f(x)) d x$$    
</center>

 where $$K$$ is the _support_ of the distribution. Computing $$Z$$ is very hard, even for simple distributions, and one has to reside in better ideas than direct computation in order to sample efficiently. 



### Markov Chain Monte Carlo

A solution to this problem brings the Markov Chain Monte Carlo (MCMC) Algorithm which simulates a Markov Chain with a stationary distribution identical to the target distribution. Multiple chains are developed, starting from a set of points arbitrarily chosen and sufficiently distant from each other. These chains move around randomly according to an algorithm that looks for places with a reasonably high contribution to the integral to move into next, assigning them higher probabilities. The general MCMC algorithm proceeds as follows:

1. Initialize the Markov Chain to an initial state $$x_0 \sim \pi_0$$ where $$\pi_0$$ is a (carefully) chosen initial distribution.

2. At each step $$t$$, where the sampler is at state $$x_t$$, make a proposal from a neighboring state $$\tilde x_t$$ and set $$x_{t + 1} = \tilde x_t$$ with probability equal to $$\min \left \{ 1, \frac {\pi(\tilde x_t)} {\pi(x_t)} \right \}$$ . Otherwise, set $$x_{t + 1} = x_t$$. This probability filter is called a Metropolis filter and gives the algorithm _incentive_ to move towards areas of higher density. (Note that the more general transition probability is equal to $$\min \left \{ 1, \frac {a(\tilde x_t \mid x_t) \pi(\tilde x_t)} {a(x_t \mid \tilde x_t) \pi(x_t)} \right \}$$ and $$a(. \mid .)$$ is a transition distribution between the states, which is usually taken to be symmetric). 

   

<center>
    <img src="https://upload.wikimedia.org/wikipedia/commons/5/5e/Metropolis_algorithm_convergence_example.png"><br>
    Figure. Markov Chain Monte Carlo Approximates the blue distribution with the orange distribution <a href="https://en.wikipedia.org/wiki/Markov_chain_Monte_Carlo">Source</a>
</center>

The speed of approximation of the desired distribution is called the _mixing time_ of the Markov Chain. The samples from the mixed chains can be used to perform classical Monte Carlo integration via the ergodic sum, that is for a function $$g$$ one has to calculate the quantity

<center>
    $$\int_K g(x) \pi(x) d x$$
</center>

which is approximated by the ergodic sum of $$T$$ samples

<center>
    $$\frac 1 T \sum_{t = 1}^T g(x_t)$$
</center>

where $$\{ x_t \}_{t \in [T]}$$ is a series of samples. The ergodic theorem states that for $$T \to \infty$$ the sum and the integral coincide, which gives MCMC methods a very powerful advantage application-wise (more below). Back to the algorithm, the next important question becomes

> In a continuous process how does one choose the proposal point $$\tilde x$$ given the current position of the walker at $$x$$? 

The following section partially answers this question.



### Hamiltonian Monte Carlo

The Hamiltonian Monte Carlo (HMC) algorithm introduced by Duane et al. (1987) assumes the existence of an imaginary particle with a state $$(x, v) \in \mathbb R^d \times \mathbb R^d$$ where $$x$$ is the position (the variable we want to sample from) and $$v$$ is the velocity of the particle. In most applications, the auxiliary variable $$v$$ comes from a normal distribution, that is $$v \sim \mathcal N (0, I_d)$$. The HMC algorithm aims to sample from the joint density $$\pi(x, v) = \pi(x) \pi(v \mid x)$$. The negative logarithm of the joint density defines a _Hamiltonian Function_ 

<center>
$$\mathcal H(x, v) = - \log \pi(x, v) = \mathcal K(v \mid x) + \mathcal U(x)$$    
</center>

 One can note that if $$\pi(x) = \exp(-f(x))$$, then $$\mathcal U(x) = f(x)$$. Moreover, usually the variable $$v$$ is independent of $$x$$ and thus the Hamiltonian depends on $$v$$ only. Now, starting from an initial state, the walker generates proposals via evolving Hamilton's equations, that is 

<center>
	$$\dot x = + \frac {\partial \mathcal H} {\partial v}$$
    $$\dot v = - \frac {\partial \mathcal H} {\partial x}$$ 
</center>

  It is straightforward to observe that the Hamiltonian $$\mathcal H(x, v)$$ is preserved over time, that is 

<center>
    $$\dot {\mathcal H} = \frac {\partial H}{\partial x} \dot x + \frac {\partial H}{\partial v} \dot v = - \dot x \dot v + \dot x \dot v = 0$$
</center>

since the stationary measure is proportional to $$\pi(x, v) \propto \exp(- \mathcal H(x,v))$$ one can simulate this process and then integrate out $$v$$ to keep samples from $$\pi(x)$$.  Ideally, if a computer had infinite precision, we could simulate the equations for some time and gather samples without using the Metropolis filter due to the preservation of the Hamiltonian. However, in a computer simulation one has to use a numerical integrator to solve Hamilton's equations. From now on, we assume the simple form of the Hamiltonian

<center>
    $$\mathcal H(x, v) = \frac 1 2 \| v \|^2 + f(x)$$
</center>

In a computer, the HMC equations are usually solved using the _leapfrog integrator_ more specifically the equations evolve according to the following rule

<center>
    $$\hat v = v - \frac \eta 2 \nabla f(x)$$
    $$\tilde x = x + \eta \hat v$$
    $$\tilde v = \hat v - \frac {\eta} 2 \nabla f(\tilde x)$$
</center>

The algorithm produces a proposal $$(\tilde x, \tilde v)$$ doing a half-step for the velocity term, then doing a full-step to upgrade the position and, finally, another half-step to update the velocity. Then, one uses a Metropolis filter to measure the change in the Hamiltonian, that is $$\min \{ 1, \exp(\mathcal H(x, v) - \mathcal H(\tilde x, \tilde v)) \}$$. The leapfrog integrator has an error of $$O(\eta^3)$$ per step and $$O(\eta^2)$$ globally. Note here that someone can use other ODE solvers, such as the Euler method, Runge-Kutta or the Collocation Method to solve the resulting ODE, each one with different guarantees regarding the error, the step and the mixing time of the walkers. Currently, GeomScale [supports](https://github.com/GeomScale/volume_approximation/pull/90) the following solvers for HMC as part of GSoC 2020

* Euler Method.
* Runge-Kutta Method.
* Leapfrog Method.
* Collocation Method (both for the differential and the integral equations).
* Richardson Extrapolation.



#### Solving HMC in a truncated setting

A more challenging, from an algorithmic standpoint, setting is when the distribution $$\pi(x)$$ is truncated to a convex body. More specifically if $$\pi(x)$$ has support $$K$$ then the truncation of $$\pi$$ to $$S \subseteq K$$ is defined as

<center>
   $$\pi_S(x) = \frac {\pi(x) \mathbf 1 \{ x \in S \}} {\int_S \pi(s) ds}$$
</center>

The adjustment to the HMC equations is that in the truncated setting, is imposing reflections of the particle at the boundary $$\partial S$$ .  From a computational viewpoint, in the case of the leapfrog integrator, one has to find the point at which the trajectory between $$x$$ and $$\tilde x$$ , that is $$tx + (1 - t)\tilde x$$ for $$t \in [0, 1]$$, intersects with some boundary normal. In its simplest case, one can use the Cyrus-Beck algorithm to calculate this intersection.  







### Using the C++ API of GeomScale

 



### References

1. mc-stan Reference Manual for Hamiltonian Monte Carlo. [Source](https://mc-stan.org/docs/2_19/reference-manual/hamiltonian-monte-carlo.html).

2. TensorFlow Reference Manual for Hamiltonian Monte Carlo. [Source](https://www.tensorflow.org/probability/api_docs/python/tfp/mcmc/HamiltonianMonteCarlo). 

3. Betancourt, Michael. "A conceptual introduction to Hamiltonian Monte Carlo." *arXiv preprint arXiv:1701.02434* (2017).

4. Lee, Yin Tat, Ruoqi Shen, and Kevin Tian. "Logsmooth Gradient Concentration and Tighter Runtimes for Metropolized Hamiltonian Monte Carlo." *arXiv preprint arXiv:2002.04121* (2020).

5. Shen, Ruoqi, and Yin Tat Lee. "The randomized midpoint method for log-concave sampling." *Advances in Neural Information Processing Systems*. 2019.

6. Chevallier, Augustin, Sylvain Pion, and Frédéric Cazals. "Hamiltonian Monte Carlo with boundary reflections, and application to polytope volume calculations." (2018).

7. Duane, Simon, et al. "Hybrid monte carlo." *Physics letters B* 195.2 (1987): 216-222.

   