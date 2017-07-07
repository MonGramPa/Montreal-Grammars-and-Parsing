---
layout: section
title: Deriving a lower bound
index: 1
bibliography:
- 'variational.bib'
---

Introduction
============

The Model
---------

The model we are working with is the Adaptor Grammar model, first
introduced in @Johnson2007. Specifically, we’ll be giving some
background and fleshing out the derivations for the variational Adaptor
Grammar presented in @Cohesssn2010. If you haven’t already, read these two
papers before going further.

The Problem
-----------

Given our generative model (the adaptor grammar) and our data, we would
like to find a posterior distribution: $$P(Hypothesis \mid  Evidence)$$. Using
Bayes’ Rule, we get:
$$P(H\mid E) = \frac{P(E\mid H)P(H)}{P(E)} = \frac{P(E\mid H)P(H)}{\sum\limits_{\forall H} P(E\mid H) P(H)}$$
We call the numerator of the fraction on the right the “generative
model”. It is composed of the product of the likelihood *of the
hypothesis* ($$P(E\mid H)$$) and the prior probability of the hypothesis
($$P(H)$$). Note that the former is not a probability but a measure of how
well our hypothesis fits the data. The denominator is the “marginal
likelihood” of the data, $$P(E)$$. To find this, we need to marginalize
out (sum over) all possible hypotheses. Because the hypotheses are the
range of values for all of the latent variables in our model, this
summation is computationally intractable. In order to obtain a
posterior, we are forced to use some approximate means of finding this
denominator. Often, a sampling approach is used. However, sampling can
be very slow to converge and is not easily parallelizable across
multiple cores. The variational Bayesian approach, on the other hand,
treats the problem of finding an appropriate marginal distribution as an
optimization problem (this will be explained in more detail). 

 
-   Let $$Z$$ be our set of hidden variable collections: 
    - $$\Theta = \{\Theta_A \mid  A \in \mathcal{N}\}$$: PCFG Multinomials
    - $$\nu = \{\nu_A \mid  A \in \mathcal{M}\}$$: stick length proportions
    - $$z_a = \{z_{A,i} \mid  A \in M, i \in \{1,...\} \}$$: adapted nonterminal's subtrees 
    - $$z' = \{z_A \mid  A \in \mathcal{M}\}$$: the full tree derivations

-   Let $$\Phi$$ be the collection of all model parameters (Pitman-Yor
    parameters $$a,b$$ and Dirichlet distribution parameter $$\alpha$$.

-   Let $$X$$ be the set of observations. In the case of word
    segmentation, for example, these would be each string of unsegmented
    phonemes.

-   Note that our goal is to find $$P(Z\mid X)$$, the posterior (where $$Z$$ is
    the set of latent variables)

-   recall $$P(Z\mid X) = \frac{P(X\mid Z, \Phi)P(Z \mid  \Phi)}{\sum\limits_{\forall Z} P(X\mid Z, \Phi) P(Z\mid \Phi) } $$ 


Let’s also write out the generative model as a joint probability
distribution: 

<center>
$$\begin{aligned}
P(X\mid Z, \Phi)P(Z\mid \Phi) = P(X\mid z', z_a, \Theta, \nu, a, b, \alpha)P(Z\mid G, a,b,\alpha)= \\
P(X\mid z', z_A, G, \nu, \Theta, a, b, \alpha) \\
\times P(z' \mid  z_a, G, \nu, \Theta, a, b, \alpha) \\
\times P(z_a \mid  G, \nu, \Theta, a,b,\alpha) \\
\times P(\nu \mid  a,b)\\
\times P(\Theta\mid \alpha) \end{aligned}$$
</center>

 This shows the dependencies in the generative model. For
example, $$\Theta$$, the PCFG multinomial, only depends on $$\alpha$$, the
Dirichlet distribution hyperparameter, but $$z'$$, the full tree
derivations, depend on the whole rest of the model. These dependencies
will appear again later.

Some useful math
================

Jensen’s inequality
-------------------

-   Jensen’s inequality states that for a convex function $$f $$ and random
    variable $$X$$: $$f(\mathbb{E}[X]) \leq \mathbb{E}[f(X)] $$

-   We’re using the logs of probabilities here, so our function is
    actually concave. As it turns out, Jensen’s inequality works both
    ways, meaning we just switch the direction of the inequality:
    $$\log(\mathbb{E}[X]) \geq \mathbb{E}[\log(X)] $$

Expected value
--------------

-   note that for discrete random variables (which we are dealing with
    in this case)
    $$\mathbb{E}_q(f(x)) = \sum\limits_{\forall x} q(x)f(x) $$

Logarithms
----------

Throughout this derivation (and the variational literature as a whole)
you will see $$\log\ P(...)$$. There are various reasons to take the log
of a probability. Some of these are more mathematical (as you’ll see, it
will appear in several expanded formulas) and some are more practical
taking a logarithm allows us to transform expensive multiplication and
division into cheaper addition and subtraction, and helps us avoid very
small probabilities below the bound which a computer can accurately
represent. Recall from high school math:


-  $$\lim\limits_{n\rightarrow 0} \log\ n = -\infty $$

-  $$\log\ AB = \log\ A + \log\ B $$

-  $$\log\ \frac{A}{B} = \log\ A - \log\ B $$

Derivation of variational bound
===============================

Derivation
----------

The value we are looking to approximate is our denominator, which is the
likelihood of the data integrated over all hypotheses. Recall that our
inference problem lies in finding the denominator to the Bayesian
equation
$$P(H\mid E) = \frac{P(E\mid H)P(H)}{\int\limits_{\forall H} P(E\mid H) P(H) dH}$$
Our hypotheses in this case are possible values for the latent variables
in the model. This integral (or in this case summation) is
computationally intractable, so we use a variational approximation for
it.\
One way we can do this is by using the Kullback Leibler (KL) divergence between this
intractable integral and some variational distribution $$q$$.\

1.  Let $$q_\nu(Z)$$ be a family of variational distributions with
    variational parameter $$\nu$$.

2.  to get the marginal likelihood ($$\log\ p(X\mid \Phi)$$) we will take the
    KL divergence between $$q_\nu(Z)$$ and $$p(Z\mid X,\Phi)$$.

3.  KL divergence is given by:

<center>
$$\begin{aligned} 
D_{KL}(q_\nu (Z) \mid \mid  p(Z\mid X,\Phi)) = \mathbb{E}_q[\log\ \frac{q_\nu(Z)}{p(Z\mid X,\Phi)}] \\
 = \mathbb{E}_q [\log\ q_\nu(Z)- \log\ p(Z\mid X, \Phi)] \\
 = \mathbb{E}_q [\log\ q_\nu(Z)- \log\ \frac{p(Z,X\mid \Phi)}{p(X\mid \Phi)}] \\
 = \mathbb{E}_q [\log\ q_\nu(Z)- (\log\ p(Z,X\mid \Phi) - \log\ p(X\mid \Phi))] \\
 = \mathbb{E}_q [\log\ q_\nu(Z)] - \mathbb{E}_q [\log\ p(Z,X\mid \Phi)] + \log\ p(X\mid \Phi) 
 \end{aligned}$$
</center>

\[@blei.d:2006\] If we think about what KL divergence represents, we can
intuitively understand why it cannot be negative. From here, we can see
how minimizing this equation is the same as maximizing the lower bound
on $$\log\ p(X\mid \Phi)$$ 

<center>
$$\begin{aligned}
0 \leq \mathbb{E}_q [\log\ q_\nu(Z)] - \mathbb{E}_q [\log\ p(Z,X\mid \Phi)] + \log\ p(X\mid \Phi)\\
- \log\ p(X\mid \Phi) \leq \mathbb{E}_q [\log\ q_\nu(Z)] - \mathbb{E}_q [\log\ p(Z,X\mid \Phi)]  \\
\log\ p(X\mid \Phi) \geq \mathbb{E}_q [\log\ p(Z,X\mid \Phi)] - \mathbb{E}_q [\log\ q_\nu(Z)] \end{aligned}$$
</center>

Another way to reach this same equation is by using
Jensen’s inequality. Consider the log marginal likelihood:
$$\log\ p(x\mid \Phi) = \log\ \sum\limits_{\forall z \in \mathbf{Z}} p(x,z\mid \Phi)$$\
The sum marginalizes out the hidden variables $$z$$ in the joint
probability distribution. Picking any variational distribution $$q(z)$$ we
can multiply the by $$\frac{q(z)}{q(z)}$$:\
$$\log\ \sum\limits_{\forall z \in \mathbf{Z}} ( p(x,z\mid \Phi) * \frac{q(z)}{q(z)} ) = \log\ \sum\limits_{\forall z \in \mathbf{Z}} q(z) \frac{ p(x,z\mid \Phi) }{q(z)}$$
Jensen’s inequality implies
$$\log\ \sum\limits_{\forall z \in \mathbf{Z}} q(z) \frac{p(x,z\mid \Phi) }{q(z)}  \geq \sum\limits_{\forall z \in \mathbf{Z}} q(z) \log\ \frac{ p(x,z\mid \Phi) }{q(z)}$$
Recall that $$\log\ (\frac{x}{y})= \log\ (x) - \log\ (y)$$. So this
equation can be broken into: <br/>
<center>
$$\begin{aligned}
\sum\limits_{\forall z \in \mathbf{Z}} q(z) \log\ \frac{ p(x,z\mid \Phi) }{q(z)}= \sum\limits_{\forall z \in \mathbf{Z}} q(z) (\log p(x,z\mid \Phi) - \log q(z)) = \\
 \sum\limits_{\forall z \in \mathbf{Z}} q(z) \log p(x,z\mid \Phi) - \sum\limits_{\forall z \in \mathbf{Z}}  q(z)\log\ q(z) =  \\
 \sum\limits_{\forall z \in \mathbf{Z}} q(z) \log p(x,z\mid \Phi) + \mathcal{H}(q)\\
\end{aligned}$$
</center>
where 
<center>
$$\begin{aligned}
\mathcal{H}(q) =  - \sum\limits_{\forall z \in \mathbf{Z}}  q(z)\log\ q(z) \end{aligned}$$
</center>

[@blei2017variational]. This first term is of the form of our expected
value definition in §1.3, so our equation becomes:

<center>
$$\begin{aligned} \log\ p(x\mid \Phi) \geq \mathbb{E}_q[\log\ p(x,z\mid \Phi)] + \mathcal{H}(q) \end{aligned}$$
</center>

This derivation yields an important fact: 
<center>
$$\begin{aligned}
\log\ p(X\mid \Phi) - KL(q(Z) \mid \mid  p(Z\mid X, \Phi)) = \mathbb{E}_q[\log\ p(z,x \mid  \Phi)] + H(q) \end{aligned}$$
</center>
From this equation, we can see why minimizing KL divergence gives us the
best possible value for our marginal likelihood.

Applying variational methods to the model
-----------------------------------------

Recall our full joint probability distribution for our generative model
from §1.2: 
<center>
$$\begin{aligned}
P(X\mid Z, \Phi)P(Z\mid \Phi) = P(X\mid z', z_a, \Theta, \nu, a, b, \alpha)P(Z\mid G, a,b,\alpha)= \\
P(X\mid z', z_A, G, \nu, \Theta, a, b, \alpha)\\
\times P(z' \mid  z_a, G, \nu, \Theta, a, b, \alpha) \\
\times P(z_a \mid  G, \nu, \Theta, a,b,\alpha) \\
\times P(\nu \mid  a,b)\\
\times P(\Theta\mid \alpha) \end{aligned}$$ </center>

Recall that doing variational inference requires inference only over our
latent variables. This is because given a full tree derivation $$z'$$, the relationship between $$z'$$ and $$X$$ is deterministic in the sense that the yields of the derivations in $$z'$$ map exactly to the strings in $$X$$.
However, to get to $$z'$$ we need to do variational inference on
$$P(Z\mid G, a,b,\alpha)$$. This part expands to: 

<center>
$$\begin{aligned}
P(z' \mid  z_a, G, \nu, \Theta, a, b, \alpha) \\
\times P(z_a \mid  G, \nu, \Theta, a,b,\alpha) \\
\times P(\nu \mid  a,b)\\
\times P(\Theta\mid \alpha)
\end{aligned}$$
</center>

So going back to our equation for the marginal probability, we need to find 
<center>
$$\begin{aligned}
\mathbb{E}_q [\log\ P(Z\mid G, a, b, \alpha)] + H(q) \\
 = \mathbb{E}_q [\log\ P(z' \mid  z_a, G, \nu, \Theta, a, b, \alpha)] \\
 + \mathbb{E}_q [\log\ P(z_a \mid  G, \nu, \Theta, a,b,\alpha)] \\
 + \mathbb{E}_q [\log\ P(\nu \mid  a,b)] \\
 + \mathbb{E}_q [\log\ P(\Theta\mid \alpha)] + H(q)\\
 \end{aligned}$$
 </center>

If we can assume that the data contains only strings that can be parsed by the grammar, we do not need to consider G as a parameter, since the rules for producing and for parsing strings are the same. We also know from the definition of our model which
hidden variables depend on which model parameters: $$\Theta$$ depends only
on $$\alpha$$ since each $$\Theta_A \sim Dir(\alpha_A)$$. $$\nu$$ depends on
$$a = \{a,b\}$$ ($$\nu_a \sim Beta(a_A,b_A)$$), the adapted nonterminal subtrees
$$z_A$$ depend on both $$\Theta$$ and $$\nu$$ and the full tree derivation
depends on $$\nu$$. We can use this information to rewrite the equation
more accurately: 

<center>
$$\begin{aligned}
\mathbb{E}_q [\log\ P(Z\mid G, a, b, \alpha)] + H(q) \\
 = \mathbb{E}_q [\log\ P(z' \mid  \nu)] \\
 + \mathbb{E}_q [\log\ P(z_a \mid   \nu, \Theta )] \\
 + \mathbb{E}_q [\log\ P(\nu \mid  a,b)] \\
 + \mathbb{E}_q [\log\ P(\Theta\mid \alpha)]  + H(q)\\
\end{aligned}$$
</center>

Finally, we know from our definition of
$$\Theta,\ \nu,\ z_A,$$ and $$z$$ in §1.1 that we can decompose them into
their subscripted constituents. This gives us: 
<center>
$$\begin{aligned}
= \sum\limits_{A\in\mathcal{M}}( \mathcal{H}(q) + \mathbb{E}_q[\log\ p(x, \Theta \mid  \alpha)] \\
+ \mathbb{E}_q[\log\ p(x, \nu \mid  a) ]\\
+ \mathbb{E}_q[\log\ p(x, z_A \mid  \nu, \Theta)] \\
+ \mathbb{E}_q[\log\ p(x, z' \mid  \nu)] )\\
\end{aligned}$$
</center>


<center>
$$\begin{aligned}
= \mathcal{H}(q) + \sum\limits_{A\in\mathcal{M}} \mathbb{E}_q[\log\ p(x, \Theta \mid  \alpha)] 
+ \sum\limits_{A\in\mathcal{M}} \mathbb{E}_q[\log\ p(x, \nu \mid  a) ]
+ \sum\limits_{A\in\mathcal{M}} \mathbb{E}_q[\log\ p(x, z_A \mid  \nu, \Theta)] 
+ \sum\limits_{A\in\mathcal{M}} \mathbb{E}_q[\log\ p(x, z' \mid  \nu)] 
\end{aligned}$$
</center>



