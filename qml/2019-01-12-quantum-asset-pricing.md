---
id: 1
layout: chapter
chapter: 7 
title: Quantum Asset Pricing
comments: true 
status: done 
tags: qml, other 
description: Quantum Algorithms for finance
author:
- 'Alessandro “Scinawa” Luongo'
bibliography:
- 'sample.bib'
- 'Mendeley.bib'
---

We want to use a quantum computer for trading. Iordanis during talks says that quantum machine learning is both overhyped and underestimated. I say that quantum finance is not overhyped but underestimated.

In terms of financial value, quantum can have a much bigger impact (in terms of money) in fianance than in other fields. Let me quote Jack Bogle here, the founder of Vanguard:

In recent years, annual trading in stocks ... averaged some $33 trillion. But capital formation ‐‐ that is, directing fresh investment capital to its highest and best uses, such as new businesses, new technology, medical breakthroughs, and modern plant and equipment for existing business ‐‐ averaged some $250 billion. Put another way, speculation represented about 99.2 percent of the activities of our equity market system, with capital formation accounting for 0.8 percent."[JB]

# Quantum price assets evaluation


Imagine we are given the historical values of $n$ asset for $T$ time steps. These values are stored in a matrix $\Pi_s(t)$. $s \in [n]$ $t \in [T']$.

From now on, we discretize time in intervals of $\Delta$.

##### Def: return of financial asset 
Is the percentage of profit or loss overa a time period. Let $\Delta t \in \mathbb{Z_+}. The return of the asset $s$ at time $t$ is:

y_s(t) := \frac{\pi\_s(t)-\pi\s_(t-\Delta t)}{\pi\s_(t-\Delta t)}



### Step 1: Storing returns as amplitudes

We are given access to the following oracle: a QRAM that stores the entries of the matrix $\Pi$.

$$ \ket{s}\ket{t} \to \ket{s}\ket{t}\ket{\Pi_s(t)} $$

From this it is easy (and deterministic) to compute the return of a given financial asset (you just need to apply quantum arthmetics:

$$ $$

Finally you can get:

$$ \ket{s}\ket{t} \to \ket{s}\ket{t}\ket{y_s(t)} $$

From this, we can use the same 'ol trick to perform a controlled rotation over an ancilla register and perform amplitude amplification on one  of the states. Formally:

$$\ket{y\_s(y)} \mapsto \ket{y\_s(t)}( \sqrt{1-\delta^2y\_s(t)\ket{0} + \deta y\_s(y)\ket{1}) $$

Where $\deta$ is chosen wisely so to have all the norms equal to 1. This will allow us to create the state:

$$ \ket{\Chi} := \frac{1}{N\_y} \sum\_t\sum\_s y\s(y)\ket{s}\ket{t} = \frac{1}{N\_y} \sum\_t\sum\_s \\norm{y_s(y)}\ket{s}\ket{t} $$


With $N\_y = $.


There's a caveat: performing amplitude is not a deterministic step, so we need to calculate the probability of the ancilla being measured. Luckly, this is $P_x = \delta^2 \sum\_{s,t} y\_s(t)^2/TN $.
Under the assumption that most of the returns are around the same order of magnitude, than we succeed with constant probability. 

By passing through the Fourier basis (i.e. applying an Hamadard gate on the index register) it is possible to compute the expected value of a superposition. Let me be more clear:

$$ \ket{\Xhi} \mapsto \ket{}    = \ket{R}$$

The success probability is given by: $p(0):=$


## Algorithm 1
Sample from $\ket{R}$. The probability of a stock is:

$$ p(s) = (\frac{\sum\_=1^T y\_s(t)}{|y'|})^2 $$.

Thus

### Variations around Algorithm 1
what you can do, instead of sampling is the following:

- Perform amplitude estimation of certain assets and compare them

- Perform tomography. Since you know the unitary which leads to the creation of $R$, you can apply [tomography](tomogrpahy) algorithm: The run time to get the whole vector is $s\log s \ \epsilon^2 $. Where $\epsilon$ si the relative error in your approximation. 

Obviously this alorithm is very trivial and it doesn't make much sense in practice, since calculating these values is linear in the dimension of the problem. Also, as we will see in 2 section, we can also assume to have the expected returns already precomputed and stored into a QRAM. 


### Covariance matrix as density matrix





## Portfolio optimization. 
In this algorihm we want to solve a constrained optimization problem. Note that this example lends very well to show the common duality in many constrained optimization problems:
- either you aim for a specific return while minimizing the risk, or
- you maximize the return up to a certain amount of risk you have decided forehead. 

Let the total wealth that we want to invest be $\Xi$, while the current portfolio of the investor is holding at current time. We want to design an algorithm to solve a *quality-constrained quadratic program*:

$$ \min_{\vec{w} \vec{w}\Sigma\vec{w} $$
 
$$\text{s.t.} \vec{R}^T \vec{w} = \mu $$

$$ \vec{\Pi}^T\vec{w} = \Xi $$ 


Usually, constrained optimization problems are solving using a techniques called Lagrange multipliers. (Here you go, if you don't know how to Lagrange Multipliers works, I have selected a nice list of [one]() [two]() [three]() articles for you).

Long story short, it is well known how to reduce a Lagrange Multipliers problem into a linear system of equaiton $M\vec{x}=\vec{b}$.






The original algorithm for performing matrix inversion uses the sample-based Hamiltonian Simulation technique. ([here](hamiltonian) if you want to know more about Hamiltonian Simulation). It is basically a HS technique where, instead of having access to the Hamiltonian as "usual" (i.e. componentwise), we are given multiple copies of a quantum state $\rho$ which is proportional to the Hamiltonian we have to simulate. In this way, by using partial application of the swpa test, it is possible to "apply" the Hamiltonian specified by $\rho$ to another quantum state $\sigma$. 



### What to do with the resulting quantum state 

- perform tomography: this will cost $n\log n \ \epsilon^2$,so it wont give us exactly an exponential speedup.

- measure the risk of the optimal portfolio

- compare two portfolios in trace distance

- Measure projectors: morally this can be interpreted as obtaining the weighs of the portfolio in certain sector of the economy (i.e. imagine you do a projector for all the IT companties.)

- sample. This approach makes sense only in the case where it is expected that a few companies will capture all the value (or most of) of the market.

- use other algorithms :) 



