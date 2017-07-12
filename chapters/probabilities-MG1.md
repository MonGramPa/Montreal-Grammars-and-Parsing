---
layout: chapter
title: Notes on probabilistic MG parser
description: Adding probabilities to MG model rules
---

## Probabilities and MG parser
by Eva P. July 12, 2017 with parts taken from _Notes on Harkema (2001:§4)_

This is a modified version of Harkema (2001:§4)'s Bottom-Up parser, which allows for selector features to indicate direction of merge with selectee AND incorporates inside probabilities for each step of a derivation.

## Modified GM1

- **Axioms.** For every $$\lambda \in Lex$$ where $$\lambda$$'s syntactic features are $$\gamma,\delta \in Cat^* $$ and $$\lambda$$'s phonetic features cover $$w_{i+1}\cdots w_{j}$$ of $$w$$, let the following be an axiom: $$[(i,j): \gamma ] $$ .
- **Goals.** $$[(0, n): $$`c`$$], [(0, n): $$`c`$$] $$
- **Rules.** Let $$0 \leq i, j, v, w \leq n; 0 \leq k, l \leq n ; 0 \leq p,q \leq 1 $$.
  * Merge
    - merge-L (merge non-mover to left)

      $$ \frac{([(i,j):=x^{L}\gamma, \alpha_1,\ldots,\alpha_k],p) ([(j,v):x,\tau_1,\ldots,\tau_l],q)}{([(i,w):\gamma, \alpha_1,\ldots,\alpha_k,\tau_1,\ldots,\tau_l],r+=P(merge-L)\cdot p \cdot q)} $$

    - merge-R (merge non-mover to right)

      $$ \frac{([(i,j):=x^{R}\gamma, \alpha_1,\ldots,\alpha_k],p) ([(v,i):x,\tau_1,\ldots,\tau_l],q)}{([(v,j):\gamma, \alpha_1,\ldots,\alpha_k,\tau_1,\ldots,\tau_l],r+=P(merge-R)\cdot p \cdot q)} $$

    - merge-m (merge mover)

      $$ \frac{([(i,j):=x^{L/R}\gamma, \alpha_1,\ldots,\alpha_k],p) ([(v,w):x\delta,\tau_1,\ldots,\tau_l],q)}{([(i,j):\gamma, \alpha_1,\ldots,\alpha_k,(v,w):\delta,\tau_1,\ldots,\tau_l],r+=P(merge-m)\cdot p \cdot q)} $$


  * Move (in progress)
    - move-1 (move to final landing position)

      $$\frac{(([(i,j):+y\gamma,\alpha_1,\ldots,\alpha_k,(v,i):-y,\tau_1,\ldots,\tau_l],r)}{([(v,j):\gamma,\alpha_1,\ldots,\alpha_k,\tau_1,\ldots,\tau_l],r+= P(move-1) \cdot r)} $$

    - move-2 (mover with multiple Licensee features)
    
      $$\frac{(([(i,j):+y\gamma,\alpha_1,\ldots,\alpha_k,(v,w):-y\delta,\tau_1,\ldots,\tau_l],r)}{([(i,j):\gamma,\alpha_1,\ldots,\alpha_k,(v,w):\delta,\tau_1,\ldots,\tau_l],r+= P(move-1) \cdot r)} $$
