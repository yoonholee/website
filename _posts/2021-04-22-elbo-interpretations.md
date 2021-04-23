---
layout: distill
title: Better Interpretations of the ELBO
description: A generalized KL divergence, Helmholtz free energy, and bits-back coding
date: 2021-04-22

authors:
  - name: Yoonho Lee
    url: /
    affiliations:
      name:

bibliography: 2018-12-22-distill.bib

_styles: >
  .my-img {
    text-align: center;
    padding: 1.5rem 0rem;
  }

  d-article p, d-article ul, d-article ol, d-article blockquote {
    margin: 0.5rem 0;
  }
---

Variational Bayesian methods optimize a quantity called the Evidence Lower BOund (ELBO).
I think the standard way of motivating and deriving this objective fails to capture what it is and why we should care.

Here is the standard setup and derivation of the ELBO.
Unobserved variables $$z$$ generate observed data $$x$$ according to $$p(x|z)$$.
We use a distribution $$q(z)$$ to approximate the posterior $$p(z|x) \propto p(z)p(x|z)$$.
We can get a lower bound of the evidence using Jensen's inequality:

$$ \begin{align}
\log p(x) &= \log \mathbb{E}_{z \sim q(z)} \left[\frac{p(x, z)}{q(z)} \right] \\
&\geq \mathbb{E}_{z \sim q(z)} \left[ \log \frac{p(x, z)}{q(z)} \right] \triangleq \mathrm{ELBO}.
\end{align} $$

A natural question is: why is this called "the" evidence lower bound?
There are infinitely many quantities that lower bound the evidence.
What is the significance of this one?

In this post, I will motivate the ELBO from three different viewpoints. 
The ELBO directly arises as a quantity of interest in each interpretation, better motivating its use as an objective.


### KL Divergence for Unnormalized Densities

Let $$f(z)$$ be an unnormalized density function, meaning that $$f(z) \geq 0$$ everywhere and $$0 < \int f(z) dz < \infty$$.
Normalizing this function to integrate to $$1$$ would result in a probability density function:

$$
p_f(z) 
\triangleq \frac{f(z)}{\int f(z) dz}
= \frac{f(z)}{Z}.
$$

Note that the KL divergence between $$q(z)$$ and $$p_f(z)$$ can be expressed using $$q(z)$$ and $$f(z)$$:

$$ 
\begin{align}
\mathrm{KL} \left( q(z) \rVert p_f(z) \right)
&= \mathbb{E}_{z \sim q(z)} \left[ \log q(z) - \log p_f(z) \right] \\
&= \mathbb{E}_{z \sim q(z)} \left[ \log q(z) - \log f(z) \right] + Z \\
&\triangleq \mathrm{KL}^* \left( q(z) \rVert f(z) \right) + Z.
\end{align} 
$$

The constant $$Z$$ can be ignored if we want to minimize this w.r.t. the distribution $$q(z)$$.
Note that we have introduced the new symbol $$\mathrm{KL}^* \left( \cdot \rVert \cdot \right)$$ to signify the equation's similarity to the KL divergence.
This can be seen as a direct generalization of the KL divergence which measures the similarity between any distribution $$q(z)$$ and nonnegative function $$f(z)$$. 
$$\mathrm{KL}^* \left( q(z) \rVert f(z) \right)$$ has minimum value when $$q(z) \propto f(z)$$, just as $$\mathrm{KL} \left( q(z) \rVert p_f(z) \right) $$ has minimum value when $$q(z)=p_f(z)$$.

The ELBO is just a special case of this similarity measure where $$f_p(z) = p(x,z)$$:

$$ \begin{align}
-\mathrm{ELBO} 
= \mathbb{E}_{z \sim q(z)} \left[ \log q(z) - \log p(x, z) \right]
= \mathrm{KL}^* \left( q(z) \rVert f_p(z) \right).
\end{align} $$

### Free Energy

Think of the possible values of $$z$$ as the phase space of a physical system.
We assume $$x$$ is observed and that $$(z, x) \sim p(z)p(x|z)$$.
The statistical description of $$z$$ through Bayes rule is that $$p(z|x) \propto p(z)p(x|z)$$.
Equivalently, we can say that $$z$$ follows the Boltzmann distribution for energy function <d-footnote>
For simplicity, let inverse temperature be 1.</d-footnote>.

$$E(z) \triangleq -\log p(z) - \log p(x|z).$$ 

A fundamental result from statistical mechanics: given energy function $$E$$, the _free energy_ defined as

$$F(q(z)) \triangleq \mathbb{E}_{q(z)} \left[ E(z) \right] - H(q(z))$$ 

is minimized when $$q(z)$$ is the Boltzmann distribution w.r.t. $$E$$ <d-footnote>
Given fixed total energy. We have implicitly set the total energy level in our previous choice of temperature.</d-footnote>.
Therefore, adjusting $$q(z)$$ to minimize the free energy is equivalent to making $$q(z)$$ follow the Boltzmann distribution. 
We rearrange to recover the ELBO:

$$ \begin{align}
F(q(z)) = \mathbb{E}_{q(z)} \left[ -\log p(z) -\log p(x|z) + \log q(z) \right] = -\mathrm{ELBO}.
\end{align} $$

This interpretation of the ELBO has its roots in the concept of Helmholtz free energy in thermodynamics.
Free energy has since been shown to emerge from the principle of maximum entropy<d-cite key="jaynes1957information"></d-cite>, independently of thermodynamics.
More recently, the _free energy principle_<d-cite key="fristonFreeenergyPrincipleUnified2010a"></d-cite> casts a wide array of empirical observations of lifelike or intelligent behavior as free energy minimization.

### Description Length

We invoke a basic lemma from information theory.
Assume that a sender and receiver agree on a distribution $$p_1(x)$$ and constructed an optimal code for it.
The expected description length of a sample from $$x \sim p_2(x)$$ is

$$ 
\mathbb{E}_{x \sim p_2(x)} [-\log p_1(x)].
$$

We construct a coding scheme for transmitting observations $$x$$ generated from unobserved latent variables $$z$$.
The sender and receiver share knowledge of the generative distribution $$p(x|z)$$ along with an approximate posterior $$q(z|x)$$.
The sender uses the following coding scheme:
- Observe $$x$$ from data, sample $$z \sim p(z \vert x)$$
- Encode $$z$$ according to $$p(z)$$
- Encode $$x$$ according to $$p(x \vert z)$$
- Send encoded $$z, x$$

Naively applying the lemma above to the two parts of the code, it may seem that the expected description length for this two-part code is 

$$
\mathbb{E}_{x \sim \textrm{data}, z \sim q(z \vert x)} \left[ -\log p(z) -\log p(x|z) \right].
$$

The __Bits-back argument__ states that one can get a "refund" from this code because of duplicate information in $$z$$ and $$x$$.
The two-part code is redundant because the sender's inference model $$q(z|x)$$ is also known by the receiver.
This redundancy is the number of bits one can learn about $$z$$ from $$x$$ using $$q(z|x)$$, which is $$-\log q(z|x)$$ bits.
Therefore, the true expected description length is 

$$
\mathbb{E}_{x \sim \textrm{data}, z \sim q(z \vert x)} \left[ -\log p(z) -\log p(x|z) + \log q(z|x) \right] = -\mathrm{ELBO},
$$

so maximizing the ELBO can be seen as minimizing the expected code length of this protocol.

The bits-back argument was first introduced by Hinton and van Camp<d-cite key="hinton1993keeping"></d-cite> in the context of minimizing the description length of neural network weights.
More recently, Chen et al<d-cite key="vlae"></d-cite> use the bits-back coding interpretation of the variational autoencoder (VAE) to explain its failure modes,
and Townsend et al<d-cite key="bitsbackdeep"></d-cite> implement an efficient lossless bits-back coding scheme using a VAE.