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
$$p(x \mid \nu ) = h(x) \exp \{ \mathcal{n}^T T(x) - A(\mathcal{n})\}$$
</center>
where $$h(x)$$ is the base measure, $$\mathcal{n}$$ are the natural parameters, $$T(x)$$ are the sufficient statistics, and $$A(\mathcal{n})$$ is the log-normalizer. 

The Dirichlet Distribution
--------------------------
Using this information, let's compute the expectation in our ELBO for our Dirichlet multinomial: 

<center>
$$\begin{equation}
\begin{split} 
\mathbb{E}_q \big[ \log\ p(\Theta_A \mid \alpha_A )\big] = \\
\mathbb{E}_q \big[\log \exp \{ \begin{bmatrix} \alpha_1 -1 \\ ... \\ \alpha_k - 1 \end{bmatrix}^T \begin{bmatrix} \log(\Theta_1) \\ ... \\ \log(\Theta_k) \end{bmatrix} - \log \Gamma(\sum\limits_{j=1}^k \alpha_j) - \sum\limits_{i=1}^k \log\Gamma(\alpha_i)\} \big] = \\
\end{split}
\end{equation}$$
</center>

Since the natural parameters and log normalizer contain no terms of $$q$$, this becomes:

<center>
$$\begin{equation}
\begin{split} 
\log \Gamma(\sum\limits_{j=1}^k \alpha_j) - \sum\limits_{i=1}^k \log\Gamma(\alpha_i)  + \begin{bmatrix} \alpha_1 -1 \\ ... \\ \alpha_k - 1 \end{bmatrix}^T \begin{bmatrix} \mathbb{E}_q [\log \Theta_1] \\ ... \\ \mathbb{E}_q[\log \Theta_k] \end{bmatrix} \end{split}
\end{equation}$$

</center>

we can compute $$\mathbb{E}_q [\log \Theta_1] = \Psi (\tau^i) - \Psi (\sum\limits_{j=1}^k \tau_j)$$ (see [Blei 2003](https://endymecy.gitbooks.io/spark-ml-source-analysis/content/%E8%81%9A%E7%B1%BB/LDA/docs/Latent%20Dirichlet%20Allocation.pdf)) 