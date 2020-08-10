---
layout: post
title: "Sampling from high-dimensional (truncated) log-concave densities with GeomScale: A Gentle Introduction"
date: 2020-07-21 17:00:00 +0200
categories:
hidden: true
---

> This article serves as a reference article for my _Google Summer of Code 2020_ internship at the GeomScale organization. The project repository can be found [here](https://github.com/GeomScale/volume_approximation).

**THE ARTICLE IS A WORK IN PROGRESS AND IS UPDATED DAILY. WILL CONTAIN MISTAKES**



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
The adjustment to the HMC equations is that in the truncated setting, is imposing reflections of the particle at the boundary $$\partial S$$ .  From a computational viewpoint, in the case of the leapfrog integrator, one has to find the point at which the trajectory between $$x$$ and $$\tilde x$$ , that is $$tx + (1 - t)\tilde x$$ for $$t \in [0, 1]$$, intersects with some boundary, that corresponds to the smallest $$t_0 \in [0, 1]$$ such that the point $$t_0 x + (1 - t_0) \tilde x \in \partial S$$. The resulting boundary point is used as a pivot point for the reflection of the position and the velocity term. In its simplest case, one can use the Cyrus-Beck algorithm to calculate this intersection. When the trajectory between the initial point and the proposed point is more complicated, for example in the collocation method, one should reside in Newton-based methods (for instance the [MPSolve](https://github.com/robol/MPSolve) package for polynomial curves) or interior-point methods (using COIN-OR IPOPT) in case the problem is approached from a non-linear optimization perspective or a transform-based approach (such as the Chebyshev transform). 

## Using the API

Currently, the project comes out with two API flavours: C++ and R. While the R interface will work better for beginner users and simpler applications, the C++ interface is in general faster (but harder to program). Both APIs are based on the same philosophy: One needs to define a functor for the negative log-probability $$f(x)$$ and  its negative gradient $$- \nabla f(x)$$ so that the HMC/ULD samplers can operate using this oracle information. For the R API we provide the following simple [example](https://raw.githubusercontent.com/papachristoumarios/volume_approximation/log-concave-samplers-temp/examples/logconcave/simple_hmc.r) which showcases the straightforward use of the R API to sample. All the user needs to do is to define the following functions

```R
f <- function(x) {
	# Return negative log-probability
}

grad_f <- function(x) {
	# Return negative log-probability gradient
}
```

which are given (and wrapped) by the C++ back-bone through the `sample_points` function. For more information on how to use the R interface of the project, we redirect the interested reader to [this tutorial](https://vissarion.github.io/tutorials/volesti_tutorial_pydata.html). 

For the C++ API we provide [this example](https://raw.githubusercontent.com/papachristoumarios/volume_approximation/log-concave-samplers-temp/examples/logconcave/simple_hmc.cpp) for operation. In this case we define the `CustomFunctor` super-class which contains three sub-classes:

* The `GradientFunctor` which is a functor responsible for returning all the derivatives of the HMC ODE, which is considered to have the general form of 
<center>
    $$\frac {d^n x}{dt^n} = F(x, t)$$
</center>
which in the case of HMC has $$n = 2$$ and returns the pair $$(v, - \nabla f(x))$$ using the index counter after transforming the  higher-order ODE to a first-order ODE in a higher-dimensional space. The same functor definitions can be used to define higher-order ODEs (for other applications) via the same transformation:

<center>
    $$\dot x_i = \begin{cases} F(x_1, t) & i = n \\ x_{i + 1} & 1 \le i \le n - 1 \end{cases}$$
</center>

These ODEs can also be restricted to a Cartesian product of domains $$K_1, \dots, K_n$$ (which in the case of HMC is $$K \times \mathbb R^d \subseteq \mathbb R^d \times \mathbb R^d$$).  

* `FunctionFunctor` class is a functor that returns $$f(x)$$ with the `operator()` method
* The `parameters` class which holds any required parameters and must contain the derivative order of the oracle.



Below, we give the reader a taste of the results using the function $$f(x) = \| x \|^2 + \mathbf 1^T x$$ which has a mode at $$x = - \frac 1 2 \mathbf 1$$ in $$d = 1$$  dimensions using HMC.



## Scaling

VolEsti is able to scale efficiently to multiple dimensions and compete with TensorFlow and mc-stan. Below we showcase an comparison of drawing $$N = 1000$$ samples using the Leapfrog method in VolEsti, mc-stan, and Tensorflow for a range of dimensions $$d \in \{1, \dots 100 \} $$. We provide a semilog-y plot of the ETA as a function of the dimensions. As we observe VolEsti is significantly faster than its counterparts from 1000 to 100 times, for a large number of dimensions in the  truncated case. We have used the density $$f(x) = (x + \mathbf 1)^T x$$ with a gradient of $$\nabla f(x) = x + \mathbf 1$$, where $$\mathbf 1$$ is the $$d$$-dimensional vector with all entries equal to 1. We also compare how VolEsti scales when truncation is imposed. More specifically, we use the same density negative log-probability of $$f(x) =  (x + \mathbf 1)^T x$$, defined either on $$\mathbb R^d$$ (un-truncated setting) or to the $$d$$-dimensional cube $$\mathbb H_d = \{ x \in \mathbb R^d \mid \| x \|_{\infty} \le 1  \}$$. The comparisons are compiled to a Colab notebook [here](https://colab.research.google.com/drive/104i-N1Na0nsj_zEK5l0d-VHnAOw67hXt?usp=sharing).

<center>
    <img src="https://github.com/papachristoumarios/papachristoumarios.github.io/raw/master/_posts/figures/performance.png">
    Figure. Performance of VolEsti compared to TensorFlow and mc-stan HMC algorithms
</center>





### References

1. mc-stan Reference Manual for Hamiltonian Monte Carlo. [Source](https://mc-stan.org/docs/2_19/reference-manual/hamiltonian-monte-carlo.html).

2. TensorFlow Reference Manual for Hamiltonian Monte Carlo. [Source](https://www.tensorflow.org/probability/api_docs/python/tfp/mcmc/HamiltonianMonteCarlo). 

3. Betancourt, Michael. "A conceptual introduction to Hamiltonian Monte Carlo." *arXiv preprint arXiv:1701.02434* (2017).

4. Lee, Yin Tat, Ruoqi Shen, and Kevin Tian. "Logsmooth Gradient Concentration and Tighter Runtimes for Metropolized Hamiltonian Monte Carlo." *arXiv preprint arXiv:2002.04121* (2020).

5. Shen, Ruoqi, and Yin Tat Lee. "The randomized midpoint method for log-concave sampling." *Advances in Neural Information Processing Systems*. 2019.

6. Chevallier, Augustin, Sylvain Pion, and Frédéric Cazals. "Hamiltonian Monte Carlo with boundary reflections, and application to polytope volume calculations." (2018).

7. Duane, Simon, et al. "Hybrid monte carlo." *Physics letters B* 195.2 (1987): 216-222.

   