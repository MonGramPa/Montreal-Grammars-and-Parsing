---
layout: chapter
title: Variational Math Derivation
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
$$P(X\mid Z, \Phi)P(Z\mid \Phi) = P(X\mid z', z_a, \Theta, \nu, a, b, \alpha)P(Z\mid G, a,b,\alpha)= $$ <br/>
$$P(X\mid z', z_A, G, \nu, \Theta, a, b, \alpha)$$ <br/>
$$\times P(z' \mid  z_a, G, \nu, \Theta, a, b, \alpha) $$ <br/>
$$\times P(z_a \mid  G, \nu, \Theta, a,b,\alpha) $$ <br/>
$$\times P(\nu \mid  a,b)$$ <br/>
$$\times P(\Theta\mid \alpha)$$


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


-  .$$\lim\limits_{n\rightarrow 0} \log\ n = -\infty $$

-  .$$\log\ AB = \log\ A + \log\ B $$

-  .$$\log\ \frac{A}{B} = \log\ A - \log\ B $$

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


$$ D_{KL}(q_\nu (Z) \mid \mid  p(Z\mid X,\Phi)) = \mathbb{E}_q[\log\ \frac{q_\nu(Z)}{p(Z\mid X,\Phi)}] $$ <br/>
$$ = \mathbb{E}_q [\log\ q_\nu(Z)- \log\ p(Z\mid X, \Phi)] $$ <br/>
$$ = \mathbb{E}_q [\log\ q_\nu(Z)- \log\ \frac{p(Z,X\mid \Phi)}{p(X\mid \Phi)}] $$ <br/>
$$ = \mathbb{E}_q [\log\ q_\nu(Z)- (\log\ p(Z,X\mid \Phi) - \log\ p(X\mid \Phi))] $$ <br/>
$$ = \mathbb{E}_q [\log\ q_\nu(Z)] - \mathbb{E}_q [\log\ p(Z,X\mid \Phi)] + \log\ p(X\mid \Phi) $$


\[@blei.d:2006\] If we think about what KL divergence represents, we can
intuitively understand why it cannot be negative. From here, we can see
how minimizing this equation is the same as maximizing the lower bound
on $$\log\ p(X\mid \Phi)$$ 


$$0 \leq \mathbb{E}_q [\log\ q_\nu(Z)] - \mathbb{E}_q [\log\ p(Z,X\mid \Phi)] + \log\ p(X\mid \Phi)$$ <br/>
$$- \log\ p(X\mid \Phi) \leq \mathbb{E}_q [\log\ q_\nu(Z)] - \mathbb{E}_q [\log\ p(Z,X\mid \Phi)]  $$ <br/>
$$\log\ p(X\mid \Phi) \geq \mathbb{E}_q [\log\ p(Z,X\mid \Phi)] - \mathbb{E}_q [\log\ q_\nu(Z)]$$


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

$$\sum\limits_{\forall z \in \mathbf{Z}} q(z) \log\ \frac{ p(x,z\mid \Phi) }{q(z)}= \sum\limits_{\forall z \in \mathbf{Z}} q(z) (\log p(x,z\mid \Phi) - \log q(z)) = $$ <br/>
$$ \sum\limits_{\forall z \in \mathbf{Z}} q(z) \log p(x,z\mid \Phi) - \sum\limits_{\forall z \in \mathbf{Z}}  q(z)\log\ q(z) =  $$ <br/>
$$ \sum\limits_{\forall z \in \mathbf{Z}} q(z) \log p(x,z\mid \Phi) + \mathcal{H}(q)$$ <br/>
where $$\mathcal{H}(q) =  - \sum\limits_{\forall z \in \mathbf{Z}}  q(z)\log\ q(z)$$
[@blei2017variational]. This first term is of the form of our expected
value definition in §1.3, so our equation becomes:

<br/>
$$\log\ p(x\mid \Phi) \geq \mathbb{E}_q[\log\ p(x,z\mid \Phi)] + \mathcal{H}(q)$$
<br/>

This derivation yields an important fact: <br/>
$$\log\ p(X\mid \Phi) - KL(q(Z) \mid \mid  p(Z\mid X, \Phi)) = \mathbb{E}_q[\log\ p(z,x \mid  \Phi)] + H(q)$$ <br/>
From this equation, we can see why minimizing KL divergence gives us the
best possible value for our marginal likelihood.

Applying variational methods to the model
-----------------------------------------

Recall our full joint probability distribution for our generative model
from §1.2: 
$$P(X\mid Z, \Phi)P(Z\mid \Phi) = P(X\mid z', z_a, \Theta, \nu, a, b, \alpha)P(Z\mid G, a,b,\alpha)= $$ <br/>
$$P(X\mid z', z_A, G, \nu, \Theta, a, b, \alpha)$$ <br/>
$$\times P(z' \mid  z_a, G, \nu, \Theta, a, b, \alpha) $$ <br/>
$$\times P(z_a \mid  G, \nu, \Theta, a,b,\alpha) $$ <br/>
$$\times P(\nu \mid  a,b)$$ <br/>
$$\times P(\Theta\mid \alpha)$$

Recall that doing variational inference requires inference only over our
latent variables. This is because given a full tree derivation $$z'$$, the relationship between $$z'$$ and $$X$$ is deterministic in the sense that the yields of the derivations in $$z'$$ map exactly to the strings in $$X$$.
However, to get to $$z'$$ we need to do variational inference on
$$P(Z\mid G, a,b,\alpha)$$. This part expands to: 


$$P(z' \mid  z_a, G, \nu, \Theta, a, b, \alpha) $$ <br/>
$$\times P(z_a \mid  G, \nu, \Theta, a,b,\alpha) $$ <br/>
$$\times P(\nu \mid  a,b)$$ <br/>
$$\times P(\Theta\mid \alpha)$$

So going back to our equation for the marginal probability, we need to find <br/>
$$\mathbb{E}_q [\log\ P(Z\mid G, a, b, \alpha)] + H(q) $$ <br/>
$$ = \mathbb{E}_q [\log\ P(z' \mid  z_a, G, \nu, \Theta, a, b, \alpha)] $$ <br/>
$$ + \mathbb{E}_q [\log\ P(z_a \mid  G, \nu, \Theta, a,b,\alpha)] $$ <br/>
$$ + \mathbb{E}_q [\log\ P(\nu \mid  a,b)] $$ <br/>
$$ + \mathbb{E}_q [\log\ P(\Theta\mid \alpha)] + H(q)$$ <br/>

If we can assume that the data contains only strings that can be parsed by the grammar, we do not need to consider G as a parameter, since the rules for producing and for parsing strings are the same. We also know from the definition of our model which
hidden variables depend on which model parameters: $$\Theta$$ depends only
on $$\alpha$$ since each $$\Theta_A \sim Dir(\alpha_A)$$. $$\nu$$ depends on
$$a = \{a,b\}$$ ($$\nu_a \sim Beta(a_A,b_A)$$), the adapted nonterminal subtrees
$$z_A$$ depend on both $$\Theta$$ and $$\nu$$ and the full tree derivation
depends on $$\nu$$. We can use this information to rewrite the equation
more accurately: 

$$\mathbb{E}_q [\log\ P(Z\mid G, a, b, \alpha)] + H(q) $$ <br/>
$$ = \mathbb{E}_q [\log\ P(z' \mid  \nu)] $$ <br/>
$$ + \mathbb{E}_q [\log\ P(z_a \mid   \nu, \Theta )] $$ <br/>
$$ + \mathbb{E}_q [\log\ P(\nu \mid  a,b)] $$ <br/>
$$ + \mathbb{E}_q [\log\ P(\Theta\mid \alpha)]  + H(q)$$ <br/>


Finally, we know from our definition of
$$\Theta,\ \nu,\ z_A,$$ and $$z$$ in §1.1 that we can decompose them into
their subscripted constituents. This gives us: 


$$= \sum\limits_{A\in\mathcal{M}}( \mathcal{H}(q) + \mathbb{E}_q[\log\ p(x, \Theta \mid  \alpha)] $$<br/>
$$+ \mathbb{E}_q[\log\ p(x, \nu \mid  a) ]$$<br/>
$$+ \mathbb{E}_q[\log\ p(x, z_A \mid  \nu, \Theta)] $$<br/>
$$+ \mathbb{E}_q[\log\ p(x, z' \mid  \nu)] )$$<br/>



$$= \mathcal{H}(q) + \sum\limits_{A\in\mathcal{M}} \mathbb{E}_q[\log\ p(x, \Theta \mid  \alpha)] $$<br/>
$$+ \sum\limits_{A\in\mathcal{M}} \mathbb{E}_q[\log\ p(x, \nu \mid  a) ]$$<br/>
$$+ \sum\limits_{A\in\mathcal{M}} \mathbb{E}_q[\log\ p(x, z_A \mid  \nu, \Theta)] $$<br/>
$$+ \sum\limits_{A\in\mathcal{M}} \mathbb{E}_q[\log\ p(x, z' \mid  \nu)] $$<br/>



Deriving Variational Updates
============================

The Coordinate Ascent Algorithm
--------------------------------

The lower bound that we've derived above is a good start, but it means nothing if we cannot use it to converge on a solution. All it tells us right now is that given a certain variational distribution $$q(Z)$$ we can compute the lower bound on the log marginal likelihood. If we were incredibly lucky, we might guess this distribution on the first try and be done, but barring this astronomically unlikely event, we will need to update our variational distribution in some iterative manner. 

Mean Field Approximation 
------------------------
The first problem with finding $$q$$ is that it is a distribution over all latent variables $$Z$$. Since it is often the strong conditional relationships between latent variables that make inference intractable, we need to find a way to simplify the problem. Intuitively, $$q$$ is an approximation (recall that L(q) is a lower bound, not an equality relation). Since we plan on improving $$q$$ through updates, it turns out we can get away with breaking these interdependencies and proposing a variational distribution $$q_i(z_i)$$ for all $$z_i \in Z$$. Since we treat each variational distribution over a latent variable as independent, we get 

$$q(Z) = \prod\limits_{i} q_i(z_i)$$

This is a very powerful assumption, because it allows us to optimize each variational distribution iteratively. That is to say, while holding all other variational distributions constant, we will find the variational parameters for $$q_i(z_i)$$ that maximize the marginal likelihood. Recall that our bound on the log marginal likelihood was:

$$L(q) \geq \sum\limits_{z_i \in Z} q(Z) \log\ p(X,Z|\Phi) + H(q)$$

For the sake of concise notation, let $$q_j = q_j{Z_j}$$ and let us disregard the conditioner $$\Phi$$ for now. Let's replace $$q(Z)$$ with our approximating product:

$$L(q) \geq \sum\limits_{z_i \in Z} \prod\limits_{i} q_i(z_i) \log\ p(X,Z|\Phi) + H(q)$$

Choosing any one $$q_j(z_j)$$ to optimize (remembering also that $$Z_j$$ can be a subset of $$Z$$, not just one element), we can rewrite this as:

$$\sum\limits_{z\in Z_j} (q_j(z) \sum\limits_{z_i \in Z_{-j}} \prod\limits_{i\neq j} q_i \log\ p(X, z) ) - \sum\limits_{z\in Z_j} q_j \log\ q_j + const$$ 

where $$Z_{_j}$$ means all $$z_i \in Z$$ where $$ i\neq j$$

Let

 $$ \log\ \tilde{p}(x,z_j) = \mathbb{E}_{-j}[\log\ p(X,Z)] + const$$

where 

$$\mathbb{E}_{-j}[\log\ p(X,Z)] = \sum\limits_{z\in Z_j} \log\ p(X,Z) \prod\limits_{i\neq j} q_i $$


Then the equation above is the negative KL divergence between $$q_j(Z_j)$$ and $$\tilde{p}(X, Z_j)$$, which we can minimize. Clearly, the minimum value for this will be reached when $$q_j(Z_j) = \tilde{p}(X, Z_j)$$. Let $$q^{*}_j(Z_j)$$ be this optimal solution. Then 

$$\log\ q^{*}_j(Z_j) = \mathbb{E}_{-j}[\log\ p(X,Z)] + const$$
@bishop2006pattern

Adding back in our conditioner $$\Phi$$ (the model parameters), the right can be rewritten:
<br/>
$$\mathbb{E}_{-j}[\log\ p(X,Z | \Phi)] + const $$<br/>
$$= \mathbb{E}_{-j}[\log\ p(Z_j, Z_{-j}, X | \Phi)] + const$$<br/>
$$ = \mathbb{E}_{-j}[\log\ p(Z_j|Z_{-j},X,\Phi)p(Z_{-j}|X, \Phi)] + const$$<br/>
$$ = \mathbb{E}_{-j}[\log\ p(Z_j|Z_{-j}, X,\Phi)] +  \mathbb{E}_{-j}[\log\ p(Z_{-j}|X, \Phi)] + const$$<br/>

We're interested in updating $$q_j(Z_j)$$, but to do so we need to hold $$q_{-j}(Z_{-j})$$ constant, so we are going to generalize this formula to say:

$$ q^*_j(Z_j) \propto \exp\{\mathbb{E}_{-j}[\log\ p(Z_j|Z_{-j}, X,\Phi)]\}$$



