---
layout: chapter
title: DPHMM Model for Acoustic Unit Discovery
description: Ondel et al. (2016)
---

Based on the model described in Ondel et al. (2016).

## Generative Model

The generative model consists of a Dirichlet distribution prior over hypothetical phone-like units, each of which is modeled by of a Hidden Markov Model over time-states of that phone, with its own set of HMM parameters. Each HMM emission state is modeled by a gaussian mixture model with $$P$$ components.

The generative model, to generate a sequence of segments of length $$N$$, is defined as follows:


1. Sample a vector $$\bm{v} = v_1, v_2, ... v_M$$ of component weights, for some number of components $$M$$, from a Beta distribution.  
$$v_i = $$Beta$$(1,\gamma)$$   
where $$\gamma$$ is the concentration parameter of the Dirichlet distribution.

2. For any individual component $$i$$, let $$\theta_i$$ represent the set of HMM parameters associated with that component's HMM. (These parameters and their prior distributions are described in more detail below.) Sample $$\theta_1,...\theta_M$$ from the HMM parameter distributions.

3. For the sequence length $$N$$, sample the segment sequence $$\bm{c}=c_1,...c_N$$ from the Dirichlet distribution, $$c_i \sim \bm{\pi}$$.

4. For each $$c_i$$, sample a state path $$\bm{s_i}=s_{i,1},...s_{i,n}$$ using the corresponding HMM parameters $$\theta_i$$

5. For each state $$s_{i,j}$$, sample a Gaussian component $$m_{i,j}$$ from the mixture model.

6. For each state $$s_{i,j}$$, sample an emitted data point from $$m_{i,j}$$.


