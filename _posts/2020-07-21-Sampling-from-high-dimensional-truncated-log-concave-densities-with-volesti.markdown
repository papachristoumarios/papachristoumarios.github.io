---
layout: post
title: "Sampling from high-dimensional (truncated) log-concave densities with GeomScale"
date: 2020-07-21 17:00:00 +0200
categories:
---

> This article serves as a reference article for my _Google Summer of Code 2020_ internship at the GeomScale organization. The project repository can be found [here](https://github.com/GeomScale/volume_approximation).



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

A solution to this problem brings the Markov Chain Monte Carlo (MCMC) Algorithm which simulates a Markov Chain with a stationary distribution identical to the target distribution. The general MCMC algorithm proceeds as follows:

1.  Initialize the Markov Chain to an initial state $$x_0 \sim \pi_0$$ where $$\pi_0$$ is a (carefully) chosen initial distribution.
2. At each step $$t$$, where the sampler is at state $$x_t$$, make a proposal from a neighboring state $$\tilde x_t$$ and set $$x_{t + 1} = \tilde x_t$$ with probability equal to $$\min \left \{ 1, \frac {\pi(\tilde x_t)} {\pi(x_t)} \right \}$$ . Otherwise, set $$x_{t + 1} = x_t$$. This probability filter is called a Metropolis filter and gives the algorithm _incentive_ to move towards areas of higher density. (Note that the more general transition probability is equal to $$\min \left \{ 1, \frac {a(\tilde x_t | x_t) \pi(\tilde x_t)} {a(x_t | \tilde x_t) \pi(x_t)} \right \}$$ and $$a(. | .)$$ is a transition distribution between the states, which is usually taken to be symmetric). 

