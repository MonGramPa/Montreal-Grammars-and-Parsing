---
layout: section
title: Deriving specific updates
index: 5
use_math: true
bibliography:
- 'variational.bib'
---


Deriving Variational Updates for Specific Parts of the Model
============================================================

The Coordinate Ascent Algorithm
--------------------------------

For a review of the general form of exponential family equations as well as the coordinate ascent algorithm and mean field approximation, see [Deriving Variation Updates](deriving_q.html)

The Exponential Family
----------------------
Recall that we can write any exponential family distribution $$p(x\mid \nu)$$ :

<center>
$$p(x \mid \nu ) = h(x) \exp \big\{ \eta^T T(x) - A(\eta)\big\}$$
</center>
where $$h(x)$$ is the base measure, $$\eta$$ are the natural parameters, $$T(x)$$ are the sufficient statistics, and $$A(\eta)$$ is the log-normalizer. 

The ELBO
--------
Recall that our ELBO function is:

<center>
$$\begin{equation}\begin{split}
= \mathcal{H}(q) + \\
+ \sum\limits_{A \in \mathcal{T}} \mathbb{E}_q[\log\ p(\epsilon \mid \rho_1, \rho_2)]
+ \sum\limits_{A \in \mathcal{T}} \mathbb{E}_q[\log\ p(\Theta_{bot} \mid  \alpha_{bot}] \\
+ \sum\limits_{A\in\mathcal{M} \setminus \mathcal{T}}  \mathbb{E}_q[\log\ p(\Theta_{top} \mid  \alpha_{top}] \\
+ \sum\limits_{A\in\mathcal{M} \setminus \mathcal{T}} \mathbb{E}_q[\log\ p(\nu \mid  a, b) ]\\
+ \sum\limits_{A\in\mathcal{M}} \mathbb{E}_q[\log\ p(z_A \mid  \nu, \Theta_{bot}, \Theta_{top}, \epsilon)] \\
+ \sum\limits_{A\in\mathcal{M} \setminus \mathcal{T}} \mathbb{E}_q[\log\ p(x, z' \mid  \nu_A)] \\
\end{split}\end{equation}$$
</center>


The Dirichlet Distribution
--------------------------
Using this information, let's compute the expectation in our ELBO for our Dirichlet multinomials $$\Theta_{top}$$ and $$\Theta_{bot}$$. This part is the same for both, so for simplicity we will use a generic variable $$\Theta$$ : 

<center>
$$\begin{equation}
\begin{split} 
\mathbb{E}_q \big[ \log\ p(\Theta \mid \alpha)\big] = \\
\mathbb{E}_q \big[\log \exp \Bigg\{ \begin{bmatrix} \alpha_1 -1 \\ ... \\ \alpha_k - 1 \end{bmatrix}^T \begin{bmatrix} \log(\Theta_1) \\ ... \\ \log(\Theta_k) \end{bmatrix} - \log \Gamma(\sum\limits_{j=1}^k \alpha_j) - \sum\limits_{i=1}^k \log\Gamma(\alpha_i)\Bigg\} \big] = \\
\end{split}
\end{equation}$$
</center>

Since the natural parameters and log normalizer contain no terms of $$q$$, this becomes:

<center>
$$\begin{equation}
\begin{split} 
\log \Gamma(\sum\limits_{j=1}^k \alpha_j) - \sum\limits_{i=1}^k \log\Gamma(\alpha_i)  + \begin{bmatrix} \alpha_1 -1 \\ ... \\ \alpha_k - 1 \end{bmatrix}^T \begin{bmatrix} \mathbb{E}_q [\log \Theta_1] \\ ... \\ \mathbb{E}_q[\log \Theta_k] \end{bmatrix} =  \\ 
\log \Gamma(\sum\limits_{j=1}^k \alpha_j) - \sum\limits_{i=1}^k \log\Gamma(\alpha_i) + \sum\limits_{i=1}^k (\alpha_i -1) ( \mathbb{E}_q[\log \Theta_i] )
\end{split}
\end{equation}$$
</center>

Because $$\mathbb{E}_q [\log \Theta_1] = \frac{\delta}{\delta \eta} A(\eta)$$ we can compute $$\mathbb{E}_q [\log \Theta_1] = \Psi (\tau^i) - \Psi (\sum\limits_{j=1}^k \tau_j)$$  (see [Blei 2003](https://endymecy.gitbooks.io/spark-ml-source-analysis/content/%E8%81%9A%E7%B1%BB/LDA/docs/Latent%20Dirichlet%20Allocation.pdf)). Rewriting our our expectation with respect to this result, we get:

<center>
$$\begin{equation}
\begin{split}
\log \Gamma(\sum\limits_{j=1}^k \alpha_j) - \sum\limits_{i=1}^k \log\Gamma(\alpha_i) + \sum\limits_{i=1}^k (\alpha_i -1) \big(\Psi (\tau^i) - \Psi (\sum\limits_{j=1}^k \tau_j)\big)
\end{split}
\end{equation}$$
</center>

The Beta Distribution
---------------------
Similarly, we can calculate the expectation of the Beta distribution on both $$\epsilon$$ and $$\nu$$. We'll do the update for $$\nu$$, but it will identical for $$\epsilon$$. 

The exponential family expansion for $$\nu$$ is: 

<center>
$$\begin{equation}
\log\  p(\nu | a, b) =\log\ \exp \Bigg\{ \begin{bmatrix}a -1 \\ b-1 \end{bmatrix}\begin{bmatrix}  \ln x \\ \ln (1-x)  \end{bmatrix} - \ln \Gamma(a) + \ln \Gamma(b) - \ln \Gamma(a + b)\Bigg\}
\end{equation}$$
</center>

recall that $$\nu$$ is actually a collection of stick proportions ($$\nu_1, ... \nu_k$$) so this becomes 

<center>
$$\begin{equation}
\begin{split}
\sum\limits_{i=1}^{N_a} \mathbb{E}_q \Bigg[(a-1)(\ln\ \nu_{A,i}) + (b-1)(\ln\ (1-\nu_{A,i}))  - \ln\Gamma(a) - \ln\Gamma(b) + \ln\Gamma(a+b)\bigg] = \\
\sum\limits_{i=1}^{N_a} a \big(\Psi(\gamma^2_{A,i}) - \Psi (\gamma^1_{A,i} + \gamma^2_{A,i}) \big) + ib \big( \Psi(\gamma^2_{A,i}) - \Psi (\gamma^1_{A,i} + \gamma^2_{A,i}) \big) + \log\ \Gamma(a + 1 + ib) - \log\Gamma(ib + a) - \log\Gamma(1-b)
\end{split}
\end{equation}$$
</center>


Sub-tree parses
---------------
Now let's expand the expectation of the sub-tree parses $$z_A$$:

<center>
$$\begin{equation}
\begin{split}
\mathbb{E}_q \big[ \log\ p(z_A,i | \nu, \Theta_{top}, \Theta_{bot}, \epsilon ) \big] = \sum\limits_{A \rightarrow \beta \in R_A} \tilde f(A \rightarrow \beta, S_{A,i}) \times \mathbb{E}_q [\Theta_{A \rightarrow \beta}] + \\
\sum\limits_{B \in \mathbb{U} \setminus \mathbb{T}} \sum\limits_{B \rightarrow \beta \in R_B} \tilde f(B \rightarrow \beta, S_{A,i}) \times \mathbb{E}_q [\log \Theta_{tops\ B\rightarrow \beta}] + \\
\sum\limits_{B \in \mathbb{T}} \sum\limits_{B \rightarrow \beta \in R_B} \tilde f(B \rightarrow \beta, S_{A,i}) \times \mathbb{E}_q [\log \Theta_{bot\ B\rightarrow \beta}] + \\
\sum\limits_{B \in \mathbb{M}} \sum\limits_{j=1}^{N_B} \tilde f(A \rightarrow S_{A,i}, S_{B,k}) \times \Big(\sum\limits_{i'=1}^{i-1} \mathbb{E}_q[\log\ \nu_{A,i'}] + \mathbb{E}_q [\log\ (1- \nu_{A,i})] \Big)
\end{split}
\end{equation}$$
</center>


recall that $$\mathbb{E}_q [\log \Theta_{A\rightarrow \beta }] = \Psi (\tau_{A \rightarrow \beta}) - \Psi (\sum\limits_{A\rightarrow \beta} \tau_{A\rightarrow \beta})$$ 
and $$\mathbb{E}_q [\log \nu_{A,i}] = \Psi (\gamma^1_{A,i}]) - \Psi (\gamma^1_{A,i} + \gamma^2_{A,i})$$


Expanding H(q)
--------------
Given the structure of the model, the entropy term $$H(q)$$ expands to 

<center>
$$\begin{equation}
\begin{split}
H(q) = \sum\limits_{A\in \mathcal{M}} \Big( H(q(\Theta_A)) \sum\limits_{i=1}^{N_A} H(q(\nu_{A,i}))\times H(q(z_{A,i}))\Big) \times \sum\limits_{i=1}^{n} H(q(z'_i)) \\
\end{split}
\end{equation}
$$
</center>


Because the Dirichlet and Beta distribution are commonly used, we already know the entropy terms for these distributions:

For the Dirichlet, entropy is 

<center>
$$\begin{equation}
\begin{split}
H(q) = \frac{\prod_{i=1}^{\mid R_A\mid } \Gamma(\alpha_i)}{\Gamma\bigl(\sum\limits_{i=1}^{\mid R_A\mid } \alpha_i\bigr)} - (\mid R_A\mid -\alpha_0)\psi(\alpha_0) - \sum\limits_{j=1}^{\mid R_A\mid } (\alpha_j-1)\psi(\alpha_j)
\end{split}
\end{equation}
$$
</center>

and for the Beta, entropy is:

<center>
$$\begin{equation}
\begin{split}
H(q) = \ln \frac{\Gamma(\gamma^1_{A,i})\Gamma(\gamma^2_{A,i})}{\Gamma(\gamma^1_{A,i} + \beta)} -(\gamma^1_{A,i}-1)\psi(\gamma^1_{A,i})-(\gamma^2_{A,i}-1)\psi(\gamma^2_{A,i}) 
+(\gamma^1_{A,i}+\gamma^2_{A,i}-2)\psi(\gamma^1_{A,i}+\gamma^2_{A,i}) 
\end{split}
\end{equation}
$$
</center>

Plugging this into the original formula for the entropy term, we get 

<center>
$$\begin{equation}
\begin{split}
H(q) = \sum\limits_{A\in \mathcal{M}} \Big( \frac{\prod_{i=1}^{\mid R_A\mid } \Gamma(\alpha_i)}{\Gamma\bigl(\sum\limits_{i=1}^{\mid R_A\mid } \alpha_i\bigr)} 
 - (\mid R_A\mid -\alpha_0)\psi(\alpha_0) - \sum\limits_{j=1}^{\mid R_A\mid } (\alpha_j-1)\psi(\alpha_j)  \\
  \sum\limits_{i=1}^{N_A}   \ln \frac{\Gamma(\gamma^1_{A,i})\Gamma(\gamma^2_{A,i})}{\Gamma(\gamma^1_{A,i} + \beta)}  -(\gamma^1_{A,i}-1)\psi(\gamma^1_{A,i})-(\gamma^2_{A,i}-1)\psi(\gamma^2_{A,i}) \\
+(\gamma^1_{A,i}+\gamma^2_{A,i}-2)\psi(\gamma^1_{A,i}+\gamma^2_{A,i})      \times H(q(z_{A,i}))\Big)  \times \sum\limits_{i=1}^{n} H(q(z'_i)) 
\end{split}
\end{equation}
$$
</center>



Lower bound for Beta Distribution
---------------------------------
Based on these expansions, we can form a variational lower bound for the beta distribution by grouping together all the terms from equations (4), (6), (7), and (11) which contain $$\nu$$:


<center>
$$\begin{equation}
\begin{split}
\mathcal{L}(\gamma^1_{A,i}, \gamma^2_{A,i}) = \log\Gamma(\gamma^1_{A,i}) + \log\Gamma(\gamma^2_{A,i}) - \log\Gamma(\gamma^1_{A,i} + \gamma^2_{A,i}) \\
 + (\gamma^1_{A,i} + \gamma^2_{A,i} - 2 + a_A + ib_A)\Psi(\gamma^1_{A,i} + \gamma^2_{A,i}) + (\gamma^1_{A,i} + a_A -1)\Psi(\gamma^1_{A,i}) + (\gamma^2_{A,i} + ib_A -1)\Psi(\gamma^2_{A,i}) \\
+ \sum\limits_{B \in \mathbb{M}} \sum\limits_{j=1}^{N_B} \tilde f(A \rightarrow S_{A,i}, S_{B,k}) \times \Big(\sum\limits_{i'=1}^{i-1} \mathbb{E}_q[\log\ \nu_{A,i'}] + \mathbb{E}_q [\log\ (1- \nu_{A,i})] \Big)
\end{split}
\end{equation}
$$
</center>



Updates for Beta
----------------
In order to get the updates for the variational beta Parameters $$\gamma^1, \gamma^2$$, we will take the derivative of the lower bound we just found with respect to the variable we want to update, set this to 0, and solve. 

This process yields the following updates:

<center>
$$\begin{equation}
\begin{split}
\gamma^1_{A,i} = 1- b_A + \sum\limits_{B \in \mathbb{M}} \sum\limits_{j=1}^{N_B} \tilde f(A \rightarrow S_{A,i}, S_{B,k})\\
\gamma^2_{A,i} = a_A + ib_A + \sum\limits_{j=1}^{i-1}\sum\limits_{B\in \mathcal{M}}\sum\limits_{k=1}^{N_B}  f(A \rightarrow S_{A,j}, S_{B,k})
\end{split}
\end{equation}
$$
</center>

Remaining Updates
-----------------
We can repeat this process in order to obtain the remaining updates:

<center>
$$\begin{equation}
\begin{split}

\kappa_{\cdot, A \rightarrow \beta} = \alpha_bot + \sum\limits_{A \rightarrow \beta \in R_bot} \tilde f(A \rightarrow \beta, S_{A,i}) 
\tau_{\cdot, A \rightarrow \beta} = \alpha_top + \sum\limits_{A \rightarrow \beta \in R_top} \tilde f(A \rightarrow \beta, S_{A,i}) \\
\phi_{\cdot, A\rightarrow s_{A,i})} = \Psi(\gamma^1_{A,i}) - \Psi(\gamma^1_{A,i} + \gamma^2_{A,i}) + \sum\limits_{j=1}^{i-1} \Big(\Psi(\gamma^2_{A,j}) - \Psi(\gamma^1_{A,j} + \gamma^2_{A,j}) \Big)\\
\phi_{\cdot, A\rightarrow \beta} = \Psi(\tau_{\cdot, A \rightarrow \beta}) - \Psi(\sum\limits_{\beta} \tau_{\cdot, A \rightarrow \beta})
\end{split}
\end{equation}
$$
</center>



