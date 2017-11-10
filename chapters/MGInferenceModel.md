---
layout: chapter
title: Inference model for MGs
description: Inference model using variational bayes for MGs
---

# Inference Model for MG Induction

#### Notes by Eva P., October 5, 2017

##Inference model
(For more in depth notes see Notes on Variational Math Derivation by Elias S.-E. )

Variational Bayesian approach
- appoximation of bayesian learning
- more efficient than sampling based methods (Zhai 2014)
- Given a posterior distribution:
$$p(H|E)= \frac{p(E|H)p(H)}{\sum_{\forall H} p(E|H)p(H)}$$
- simplification of constraints to approximate posterior as $$q(H)$$
- Mean-field variational Bayes : minimizing Kullback-Leibler distance, $$KL(q(H)|p(H|E))$$
- Turns out that that this is the same as maximizing the lower bound on the log likelihood of E:
    $$log p(E) \geq \mathbb(E)_q[log p(H,E)] -\mathbb(E)_q[log q(H)]$$

## MG Induction
- Let $$\mathbf{G}$$ be a MG,
$$\mathbf{G} = \{ Phon, \mathcal{F} , Merge , \theta \}$$
  - where Phon is the set of phonological features, $$\mathcal{F}$$ the set of syntactic and semantic 	features, Merge the merge rules.
  - $$\mathbb{L}$$ is a lexicon, such that $$\mathbb{L} \subseteq Phon \times \mathcal{F}$$
  - Let $$\theta(\mathbf{l})$$, where $$\mathbf{l} \in \mathbb{L}$$, be the weight of $$\mathbf{l}$$ and $$\theta_{\pi}(\beta)$$ mean $$\theta(\pi::\beta)$$, where $$\pi \in Phon$$, and $$\beta \in \mathcal{F}$$ .

- Next let:
  - $$d = \{d_{1}, d_{2}, ..., d_{n} \}$$ be the derivations of corpus
  - $$ C = \{s_{1}, s_{2}, ..., s_{n} \} $$ the sentences in the corpus

- We assume that the weights of each lexical item are drawn for a Dirichlet distribution parameterized by $$ \alpha_{\pi} $$
- The posterior we are interested in:

$$p(d, \theta | C, \alpha)= \frac{p(C | d, \theta, \alpha)p(\theta|\alpha)}{\sum_{\forall \theta, d} p(C | d, \theta, \alpha)p(\theta|\alpha)}$$

- Using the mean-field approximation method, define our variational distribution:
$$q(d, \theta| C) = q(d | C)q(\theta | C) = ( \prod_{i=1}^{n} q(d_{j} | s_{i}))( \prod_{\forall \pi} q(\theta_{\pi} | C))$$
- where $$d_{j}$$ is some derivation of  $$s_{i}$$
- Updating protocol similar to EM algorithm:
    - Update $$q(\theta | C)$$
      - $$q(\theta | C) = \prod_{\forall{\pi}} P_{D} (\theta_{\pi}, \hat{\alpha_{\pi}})$$
      - where
      $$\hat{\alpha_{\mathbf{l}}} = \alpha_{\mathbf{l}} + \sum_{i=1}^{n} \sum_{d_{j} \in \omega (s_{i})} q(d_{j} | s_{i})c( \mathbf{l} , d_{j} ) $$
      -  $$\alpha_{\mathbf{l}}$$ is a hyperparameter of Dirichlet prior, $$\omega (s_{i})$$ returns all the possible derivations for s_{i} and $$c( \mathbf{l} , d_{j} )$$ returns the counts of \pi::f in d.
\item Update by updating parameter \( \hat{\alpha_{\pi}} \)
 \end{itemize}
     \item Update \( q(d | C) \)
 \begin{itemize}
\item \( q(d | C) =  \prod_{i=1}^{n} q(d_{j}| s_{i}) \)
\item \( q(d_{i}| s_{i}) = \frac{q(d_{i}, s_{i})}{\sum_{d_{j} \in \omega (s_{i})} q(d_{i}, s_{i})} = \frac{\prod_{\mathbf{l} \in \mathbb{L}}\tau(\mathbf{l})^{c( \mathbf{l} , d_{j} )}}{\sum_{d_{j} \in \omega (s_{i})} \prod_{\mathbf{l} \in \mathbb{L}}\tau(\mathbf{l})^{c( \mathbf{l} , d_{j} )}} \)
\begin{itemize}
  \item where: \( \tau(\pi::f) = exp [ \psi(\hat{\alpha_{\pi::f}}) - \psi ( \sum_{f; \pi::f \in \mathbb{L}} \hat{\alpha_{\pi::f}}) ]  \)
  \item \( \psi \) is digamma function.
 \end{itemize}
\end{enumerate}
\item Alternate updating repeatedly to and maximize the lower bound on \(\mathcal{L}(C), which is functional of \(q(d, \theta | C)\) to find a q which approximates our posterior, giving us our inference model.
\end{itemize}
