---
id: 1
layout: chapter
chapter: 7 
title: Quantum Slow Feature Analysis
comments: true 
status: done
tags: qml, algos
description: Quantum Slow Feature Analyis is a dimensionality reduction algorithm for supervised problems, equivalent to Fisher Linear Discriminant. The classical algorithm comes from computational neurocsiences.
author:
- 'Alessandro “Scinawa” Luongo'
bibliography:
- 'sample.bib'
- 'Mendeley.bib'

---

The original Slow Feature Analysis (SFA) was originally proposed to
learn slowly varying features from generic input signals that vary
rapidly over time (P. Berkes 2005; Wiskott Laurenz and Wiskott 1999).
Computational neurologists observed long time ago that primary sensory
receptors, like the retinal receptors in an animal’s eye - are sensitive
to very small changes in the environment and thus vary on a very fast
time scale, the internal representation of the environment in the brain
varies on a much slower time scale. This observation is called *temporal
slowness principle*. SFA, being the state-of-the-art model for how this
temporal slowness principle is implemented, is an hypothesis for the
functional organization of the visual cortex (and possibly other sensory
areas of the brain). Said in a very practical way, we have some
“process” in our brain that behaves very similarly as dictated by SFA
(L. Wiskott et al. 2011).

Very beautifully, it is possible to show two reductions from two other
dimensionality reduction algorithms used in machine learning: Laplacian
Eigenmaps (a dimensionality reduction algorithm mostly suited for video
compressing) and Fisher Discriminant Analysis (a standard dimensionality
reduction algorithm). SFA can be applied in ML fruitfully, as there have
been many applications of the algorithm to solve ML related tasks. The
key concept for SFA (and LDA) is that he tries to project the data in
the subspace such that the distance between points with the same label
is minimized, while the distance between points with different label is
maximized.

### Classical SFA for classification


The high level idea of using SFA for classification is the following:
One can think of the training set as an input series
$x(i) \in \mathbb{R}^d , i \in [n]$. Each $x(i)$ belongs to one of $K$
different classes. The goal is to learn $K-1$ functions
$g_j( x(i)), j \in [K-1]$ such that the output
$ y(i) = [g_1(  x(i)), \cdots , g_{K-1}(  x(i)) ]$ is very similar for
the training samples of the same class and largely different for samples
of different classes. Once these functions are learned, they are used to
map the training set in a low dimensional vector space. When a new data
point arrive, it is mapped to the same vector space, where
classification can be done with higher accuracy.

Now we introduce the minimization problem in its most general form as it
is commonly stated for classification (P. Berkes 2005). Let
$a=\sum_{k=1}^{K} \binom{|T_k|}{2}.$ For all $j \in [K-1]$, minimize:

$$\Delta(y_j) =  \frac{1}{a} \sum_{k=1}^K \sum_{s,t \in T_k \atop s<t} \left( g_j( x(s)) - g_j( x(t)) \right)^2$$

with the following constraints:

1.  $\frac{1}{n} \sum_{k=1}^{K}\sum_{i\in T_k} g_j( x(i)) = 0 $

2.  $\frac{1}{n} \sum_{k=1}^{K}\sum_{i \in T_k} g_j( x(i))^2 = 1 $

3.  $ \frac{1}{n} \sum_{k=1}^{K}\sum_{i \in T_k} g_j( x(i))g_v( x(i)) = 0 \quad \forall v < j $

For some beautiful theoretical reasons, QSFA algorithm is in practice an
algorithm for fidning the solution of the *generalized eigenvalue
problem*: 

$$AW= \Lambda BW$$ 

Here $W$ is the matrix of the singular vectors, $\Lambda$ the diagonal matrix of singular values. For SFA $A$ and $B$ are defined as: $ A=\dot{X}^T \dot{X} $ and $B := X^TX$, where $\dot{X}$ is the matrix of the derivative of the data: i.e. for each possible elements with the same label we calculate the pointwise difference between vectors. (computationally, it suffice to sample $O(n)$ tuples fom the uniform distribution of all possible derivatives. 

It is possible to see that the slow feature space we are looking for is is spanned by the eigenvectors of $W$ associated to the $K-1$ smallest eigenvalues of
$\Lambda$.

### Quantum Slow Feature Analysis


In (Kerenidis and Luongo 2018) we show how, using a “QuantumBLAS” ( i.e.
a set of quantum algorithm that we can use to perform linear algebraic
operations), we can perform the following algorithms. The intuition
behind this algorithm is that the derivative matrix of the data can be
pre-computed on non-whitened data, like one might do classically (and
spare a matrix multiplication). Since with quantum computer we don’t
have this problem, since we know how to perform matrix multiplication
efficiently. As in the classical algorithm, we have to do some
preprocessing to our data. For the quantum case, preprocessing consist
in:

1.  Polynomially expand the data with a polynomial of degree 2 or 3

2.  Normalize and Scale the rows of the dataset $X$.

3.  Create $\dot{X}$ by sampling from the distribution of possible
    couples of rows of $X$ with the same label.

4.  Create QRAM for $X$ and $\dot{X}$

Note that all these operation are at most $O(nd\log(nd))$ in the size of
the training set, which is a time that we need to spend anyhow, even by
collecting the data classically.

To use our algorithm for classification, you use QSFA to bring one
cluster at the time, along with the new test point in the slow feature
space, and perform any distance based classification algorithm, like
QFDC or swap tests, and so on. The quantum algorithm is the following:

-   **Require** Matrices $X \in \mathbb{R}^{n \times d}$ and
    $\dot{X} \in \mathbb{R}^{n \times d}$ in QRAM, parameters
    $\epsilon, \theta,\delta,\eta >0$.\

-   **Ensure** A state $\ket{\bar{Y}}$ such that
    $ | \ket{Y} - \ket{\bar{Y}} | \leq \epsilon$, with
    $$Y = A^+_{\leq \theta, \delta}A_{\leq \theta, \delta} Z$$

1.  Create the state
    $$\ket{X} :=  \frac{1}{ {||X ||}_F} \sum_{i=1}^{n} {||x(i) ||} \ket{i}\ket{x(i)}$$
    using the QRAM that stores the dataset.

2.  (Whitening algorithm) Map $\ket{X}$ to $\ket{\bar{Z}}$ with
    $| \ket{\bar{Z}}  - \ket{Z} | \leq \epsilon $ and $Z=XB^{-1/2}.$
    using quantum access to the QRAM.

3.  (Projection in slow feature space) Project $\ket{\bar{Z}}$ onto the
    slow eigenspace of $A$ using threshold $\theta$ and precision
    $\delta$ (i.e.
    $$A^+_{\leq \theta, \delta}A_{\leq \theta, \delta}\bar{Z}$$ )

4.  Perform amplitude amplification and estimation on the register
    $\ket{0}$ with the unitary $U$ implementing steps 1 to 3, to obtain
    $\ket{\bar{Y}}$ with $| \ket{\bar{Y}} - \ket{Y}  | \leq \epsilon $
    and an estimator $ \bar{ {|| Y  ||} } $ with multiplicative error
    $\eta$.

Overall, the algorithm is subsumed in the following Theorem.

##### Theorem: Quantum Slow Feature Analysis
_Let $X = \sum\_i \sigma\_i u\_iv\_i^T \in \mathbb{R}^{n\times d}$ and its derivative matrix $\dot{X} \in \mathbb{R}^{n \log n \times d}$ stored in QRAM. Let $\epsilon, \theta, \delta, \eta >0$. There exists a quantum algorithm that produces as output a state $$ \ket{\bar{Y}}$$ with_
$$ | \ket{\bar{Y}} - \ket{A^+_{\leq \theta, \delta}A_{\leq \theta, \delta} Z} | \leq \epsilon $$
_in time_
$$\tilde{O}\left(  \left( \kappa(X)\mu(X)\log (1/\varepsilon) + \frac{  ( \mu({X})+ \mu(\dot{X}) ) }{\delta\theta} \right)
\frac{||{Z}||}{ ||A^+_{\leq \theta, \delta}A_{\leq \theta, \delta} {Z} ||} \right)$$
_and an estimator $\bar{||Y ||}$ with_
$ | \bar{||Y ||} - ||Y || | \leq \eta {||Y ||}$ _with an additional_
$$1/\eta$$ factor.

A prominent advantage of SFA compared to other algorithms is that *it
is almost hyperparameter-free*. The only parameters to chose are in the
preprocessing of the data, e.g. the initial PCA dimension and the
nonlinear expansion that consists of a choice of a polynomial of
(usually low) degree $p$. Another advantage is that it is *guaranteed to
find the optimal solution* within the considered function space
(Escalante-B and Wiskott 2012). We made an experiment, and using QSFA with a quantum classifier, we were
able to reach 98.5% accuracy in doing digit recognition: we were able to
read 98.5% among 10.000 images of digits given a training set of 60.000
digits.


### References

<div id="refs" class="references">

<div id="ref-Berkes2005pattern">

Berkes, Pietro. 2005. “Pattern Recognition with Slow Feature Analysis.”
*Cognitive Sciences EPrint Archive (CogPrints)* 4104.
[http://cogprints.org/4104/ http://itb.biologie.hu-berlin.de/\~berkes](http://cogprints.org/4104/ http://itb.biologie.hu-berlin.de/~berkes).

</div>

<div id="ref-escalante2012slow">

Escalante-B, Alberto N, and Laurenz Wiskott. 2012. “Slow Feature
Analysis: Perspectives for Technical Applications of a Versatile
Learning Algorithm.” *KI-Künstliche Intelligenz* 26 (4). Springer:
341–48.

</div>

<div id="ref-jkereLuongo2018">

Kerenidis, Iordanis, and Alessandro Luongo. 2018. “Quantum
Classification of the Mnist Dataset via Slow Feature Analysis.” *arXiv
Preprint arXiv:1805.08837*.

</div>

<div id="ref-scholarpedia2017SFA">

Wiskott, L., P. Berkes, M. Franzius, H. Sprekeler, and N. Wilbert. 2011.
“Slow Feature Analysis.” *Scholarpedia* 6 (4): 5282.
doi:[10.4249/scholarpedia.5282](https://doi.org/10.4249/scholarpedia.5282).

</div>

<div id="ref-wiskott1999learning">

Wiskott Laurenz, and Laurenz Wiskott. 1999. “Learning invariance
manifolds.” *Neurocomputing* 26-27. Elsevier: 925–32.
doi:[10.1016/S0925-2312(99)00011-9](https://doi.org/10.1016/S0925-2312(99)00011-9).

</div>

</div>
