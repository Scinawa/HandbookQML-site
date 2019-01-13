---
id: 1
layout: chapter
chapter: 7 
title: Estimating average and variance of a function with quantum computer
comments: true 
status: todo
tags: qml, tools
description: A very simple algorithm that uses amplitude estimation to evaluate faster mean and average of a function that we access through an oracle. 
author:
- 'Alessandro “Scinawa” Luongo'
bibliography:
- 'sample.bib'
- 'Mendeley.bib'

---

I  decided to write this post after reading a paper called [Quantum Risk Analysis](https://arxiv.org/abs/1806.06893) {% cite woerner2018quantum %} , by Stefan Woerner and Daniel J. Egger. Here I want to describe just the main technique employed by their algorithm (namely, how to use amplitude estimation to get useful information out of a function). In another post I will add describe more in detail the rest of the paper, which goes into technical details on how to use these techniques for solving a problem related to financial analysts.

Suppose we have a random variable $X$ described by a certain probability distribution over $N$ different outcomes, and a function $f: \\{0,\cdots N\\} \to \\{0,1\\}$ defined over this distribution. How can we use quantum computers to evaluate some properties of $f$ such as expected value and variance faster than classical computers?

Let's start by translating into the quantum realm these two mathematical bojects. The probability distribution is (surprise surprise) represented in our quantum computer by a quantum state over $n=\lceil \log N \rceil$ qubits. 
$$\ket{\psi} = \sum_{i=0}^{N-1} \sqrt{p_i} \ket{i} $$
where the probability of measuring the state $\ket{i}$ is $p_i,$ for $p_i \in [0, 1]$. Basically, each bases of the Hilbert space represent an outcome of the random variable. 

The quantization of the function $f$ is made by a linear operator $F$ acting on a new ancilla qubit as such:
$$ F: \ket{i}\ket{0} \to \ket{i}\left(\sqrt{1-f(i)}\ket{0} + \sqrt{f(i)}\ket{1}\right) $$

If we apply $F$ with $\ket{\psi}$ as input state we get:

$$ \sum_{i=0}^{N-1} \sqrt{1-f(i)}\sqrt{p_i}\ket{i}\ket{0} + \sum_{i=0}^{N-1} \sqrt{f(i)}\sqrt{p_i}\ket{i}\ket{1} $$

Observe that the probability of measuring $\ket{1}$ in the ancilla qubit is $\sum_{i=0}^{N-1}p_if(i)$, which is (w00t w00t) $E[f(X)]$. 
By sampling the ancilla qubit we won't get any speedup, but if we can now apply [amplitude estimation](https://arxiv.org/abs/quant-ph/0005055) {% cite brassard2002quantum %} to the ancilla qubit on the right, we can get an estimate of $E[F(X)]$. 

Finally, observe that:

-  if we chose $f(i)=\frac{i}{N-1}$ we are able to estimate $E[\frac{X}{N-1}]$ (which, by knowing $N$ gives us an estimate of the expected value of $X$)
- if we chose $f(i)=\frac{i^2}{(N-1)^2}$ instead, we can estimate $E[X^2]$ and using this along with the previous choice of $f$ we can estimate the variance of $X$: $E[X^2] - E[X]^2$.


See ya!


{% bibliography --cited %}
