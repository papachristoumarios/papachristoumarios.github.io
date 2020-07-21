---
layout: post
title: "Sampling from high-dimensional (truncated) log-concave densities with GeomScale"
date: 2020-07-21 17:00:00 +0200
categories:
---

> This article serves as a reference article for my _Google Summer of Code 2020_ internship at the GeomScale organization. The project repository can be found [here](https://github.com/GeomScale/volume_approximation).



## Problem Definition

A particularly interesting problem in statistics is taking samples from a density of the form $$\pi(x) \propto \exp(-f(x))$$ where $$f(x)$$ is a convex function. To give the interested reader a quick glance into the problem, consider the _logistic regression_ problem where $f(x)$ has a form of

<center>
$$f(x) = \frac 1 m \sum_{i = 1}^m \log(\exp(-y_i x_i^T \theta) + 1)$$
</center>

where $$\{x_i, y_i \}_{i =1}^m$$ is a set of $$m$$ samples.
