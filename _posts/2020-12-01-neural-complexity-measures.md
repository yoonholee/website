---
layout: distill
title: Neural Complexity Measures
description: A meta-learning framework for predicting generalization
date: 2020-12-01

authors:
  - name: Yoonho Lee
    url: /
    affiliations:
      name:

bibliography: 2018-12-22-distill.bib
---

Say we have a neural network $$f_\theta: \mathcal{X} \rightarrow \mathcal{Y}$$.
We want to predict and/or prevent overfitting, so we are often interested in measuring the complexity of the function $$f_\theta$$.
**How should we measure the complexity of a neural network?**
Some common approaches are:

- number of parameters in $$\theta$$
- norm of parameter vector $$\Vert \theta \Vert_p$$
- distance to initialization $$\Vert \theta - \theta_0 \Vert_p$$
- flatness of minima $$\Vert \nabla^2 \mathcal{L} (\theta) \Vert$$

Each of these hand-designed measures of complexity fails to capture the behavior of neural networks used in practice. 
A common feature, attributable to the fact that people designed them, is that they're simple equations defined in parameter space. 
I believe that defining a complexity measure in parameter space is the wrong approach. 
Parameter space is insanely messy and high-dimensional, and a simple equation like the ones above will likely be insufficient to capture this intricacy. 
Another concern is in calibration between models: changing the architecture or increasing the number of parameters alters the parameter space's geometry drastically, making a comparison between different models hard.

## Neural Complexity Measures 
We <d-cite key="lee2020nc"></d-cite> took a different approach:
(1) **we defined a complexity measure in function space**, and (2) **we learned this measure in a data-driven way**.
More concretely, for any given neural network, a meta-learned model predicts its generalization gap:
{% responsive_image path: assets/img/201201_nc_gap.png %} 

The generalization gap is a direct quantitative measure of the degree of overfitting. 
While most approaches attempt to find suitable proxies for this quantity, we adopt a meta-learning framework that treats the estimation of the generalization gap as a set-input regression problem.

We call this meta-learned estimator a Neural Complexity (NC) measure. 
We train NC with the following meta-learning loop:
{% responsive_image path: assets/img/201201_nc_loop.png %} 
We continually use NC as a regularizer for new task learning runs and store snapshots of these runs into a memory bank. 
NC itself is trained using minibatches of snapshots, sampled randomly from the memory bank.

In the paper, we show proof-of-concept experiments that show that an NC model can learn to predict the generalization gap in synthetic regression and real-world image classification problems. 
The trained NC models show signs of generalization to out-of-distribution learner architectures. 
Interestingly, using NC as a regularizer resulted in lower test loss than train loss.

## Conclusion
Neural Complexity (NC) is a meta-learning framework for predicting generalization, 
which has exciting implications for both meta-learning and understanding generalization in deep networks.
### Meta-Learning
We typically use meta-learning for small problems such as few-shot classification. 
Because they rely on feeding the entire dataset into the computation graph, previous meta-learning frameworks<d-cite key="finn2017model,garnelo2018neural"></d-cite> are not scalable to massive datasets. 
NC is a much more “local” meta-learning framework: a trained NC model acts like any other regularizer and runs alongside minibatch SGD. 
In the paper, we show that NC can successfully regularize relatively large tasks like CIFAR-10.
Further improvements may make NC a viable regularizer in ImageNet-scale tasks.
### Generalization in Deep Networks
This work showed that a meta-learned model could predict the generalization gap reliably enough to be used as a regularizer. We also proposed a simple way of translating regression accuracy to a generalization bound.
I think this shows the potential of meta-learning as a tool for understanding generalization in deep networks.
Further improvements to the NC framework and integration with theoretical tools for understanding generalization could be a promising way forward in this problem.