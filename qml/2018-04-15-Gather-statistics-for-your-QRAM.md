---
id: 1
layout: chapter
chapter: 7 
title: 'qramutils: gather statistics for your QRAM'
comments: true
status: done
tags: qml, tools
author:
- 'Alessandro “Scinawa” Luongo'
permalink: qramutils
description: "A set of scripts to estimate the parameters of the runtime of QRAM based algorithms"
bibliography:
- 'sample.bib'
- 'Mendeley.bib'

---

Generally, with the term QRAM people are referring to an oracle, or
generically to a unitary, that gets called with the purpose of creating
a state in a quantum circuit. This state represents some (classical)
data that you want to process later in your algorithm. More formally,
QRAM allows you to perform operations like:
$\ket{i}\ket{0} \to \ket{i}\ket{x_i}$ for $x_i \in \mathbb{R}$ for some
$i \in [n]$. This model can be used to create states proportional to
classical vectors, and allowing us to perform queries:
$\ket{i}\ket{0} \to \ket{i}\ket{x(i)}$ for $x(i) \in \mathbb{R}^d$ for
some $i \in [n]$

Querying the QRAM is assumed to be done efficiently. The running time is
expected to be polylogarithmic in the matrix dimensions, but eventually
the time complexity might polynomial in other parameters. As an example,
in QRAM described in Kerenidis and Prakash (2017)Kerenidis and Prakash
(2016)Prakash (2014) the authors stores a matrix decomposition such that
the running time of a query might depend on the Frobenius norm, or a
parametrized function, which is specific to their implementation. In
this model, the best parametrization of the decomposition might depend
on the dataset. This means that in practice, you might need to estimate
these parameters, and therefore I’ve decided to write a library for
this. Specifically, given a matrix $A$ to store in QRAM, you have to
find the value $p \in \left(0, 1 \right)$ such that it minimize the
function: $$\mu_p(A) = \sqrt{ s_{2p}(A) s_{2(1-p)}(A^T)}$$ where
$s_p(A) := max_{i \in [m]} |A|_F^p $ is the maximum $l_p$ norm to the
power of $p$ of the row vectors.

Being able to estimate parameters of a dataset might happen also with
other model of access to the data. For instance, other algorithms such
HHL uses Hamiltonian simulation, which has an access model that makes
the complexity of the algorithm depend on the sparsity.

So far qramutils analyze a given numpy matrix for the following
parameters:

-   The sparsity.

-   The conditioning number.

-   The Frobenius norm (of the rescaled matrix such that
    $0< \sigma_i < 1$).

-   The best parameter $p$ for the matrix decomposition described above.

-   Some boring and common plotting.

[Here](https://github.com/Scinawa/qramutils) you can find the
repository.

This code might be improved in many directions! For instance, I’d like
to integrate in the library the code for plotting the parameters for
various PCA dimensions and/or degree of polynomial expansion, integrate
options for dataset normalization, scaling, and maybe expand the type of
accepted input data, and so on..

Ideally, for other kind of matrices there hopefully might be other kind
matrix decompositions available and therefore there might be the need to
estimate other parameters in the future. This is where I’ll add that
code for that. :)

This is an example of usage on the MNIST dataset:

    $ pipenv run python3 examples/mnist_QRAM.py --help
    usage: mnist_QRAM.py [-h] [--db DB] [--generateplot] [--analize]
                         [--pca-dim PCADIM] [--polyexp POLYEXP]
                         [--loglevel {DEBUG,INFO}]

    Analyze a dataset and model QRAM parameters

    optional arguments:
      -h, --help            show this help message and exit
      --db DB               path of the mnist database
      --generateplot        run experiment with various dimension
      --analize             Run all the analysis of the matrix
      --pca-dim PCADIM      pca dimension
      --polyexp POLYEXP     degree of polynomial expansion
      --loglevel {DEBUG,INFO}
                            set log level

This is the output, assuming you have a folder called data that holds
the MNIST dataset.

    pipenv run python3 examples/mnist_QRAM.py --db data --analize --loglevel INFO
    04-01 22:23 INFO     Calculating parameters for default configuration: PCA dim 39, polyexp 2
    04-01 22:24 INFO     Matrix dimension (60000, 819)
    04-01 22:24 INFO     Sparsity (0=dense 1=empty): 0.0
    04-01 22:24 INFO     The Frobenius norm: 4.6413604982930385
    04-01 22:26 INFO     best p 0.8501000000000001
    04-01 22:26 INFO     Best p value: 0.8501000000000001
    04-01 22:26 INFO     The \mu value is: 4.6413604982930385
    04-01 22:26 INFO     Qubits needed to index+data register: 26.

If you want to use the library in your source code:

        libq = qramutils.QramUtils(X, logging_handler=logging)

        logging.info("Matrix dimension {}".format(X.shape))

        sparsity = libq.sparsity()
        logging.info("Sparsity (0=dense 1=empty): {}".format(sparsity))

        frob_norm = libq.frobenius()
        logging.info("The Frobenius norm: {}".format(frob_norm))

        best_p, min_sqrt_p = libq.find_p()
        logging.info("Best p value: {}".format(best_p))

        logging.info("The \\mu value is: {}".format(min(frob_norm, min_sqrt_p)))

        qubits_used = libq.find_qubits()
        logging.info("Qubits needed to index+data register: {} ".format(qubits_used))

To install, you just need to do the following:

    pipenv run python3 setup.py sdist

And then, your package will be ready to be installed as:

    pipenv install dist/qramutils-0.1.0.tar.gz

<div id="refs" class="references">

<div id="ref-kerenidis2016quantum">

Kerenidis, Iordanis, and Anupam Prakash. 2016. “Quantum Recommendation
Systems.” *ArXiv Preprint ArXiv:1603.08675*.

</div>

<div id="ref-kerenidis2017quantum">

———. 2017. “Quantum Gradient Descent for Linear Systems and Least
Squares.” *ArXiv Preprint ArXiv:1704.04992*.

</div>

<div id="ref-prakash2014quantum">

Prakash, Anupam. 2014. *Quantum Algorithms for Linear Algebra and
Machine Learning*. University of California, Berkeley.

</div>

</div>
