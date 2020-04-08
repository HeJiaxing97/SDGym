<p align="left">
<img width=15% src="https://dai.lids.mit.edu/wp-content/uploads/2018/06/Logo_DAI_highres.png" alt=“SDGym” />
<i>An open source project from Data to AI Lab at MIT.</i>
</p>

[![Development Status](https://img.shields.io/badge/Development%20Status-2%20--%20Pre--Alpha-yellow)](https://pypi.org/search/?c=Development+Status+%3A%3A+2+-+Pre-Alpha)
[![Travis](https://travis-ci.org/sdv-dev/SDGym.svg?branch=master)](https://travis-ci.org/sdv-dev/SDGym)
[![PyPi Shield](https://img.shields.io/pypi/v/sdgym.svg)](https://pypi.python.org/pypi/sdgym)
<!--[![Coverage Status](https://codecov.io/gh/sdv-dev/SDGym/branch/master/graph/badge.svg)](https://codecov.io/gh/sdv-dev/SDGym)-->
[![Downloads](https://pepy.tech/badge/sdgym)](https://pepy.tech/project/sdgym)

# SDGym - Synthetic Data Gym

* License: [MIT](https://github.com/sdv-dev/SDGym/blob/master/LICENSE)
* Development Status: [Pre-Alpha](https://pypi.org/search/?c=Development+Status+%3A%3A+2+-+Pre-Alpha)
<!--* Documentation: https://sdv-dev.github.io/SDGym/-->
* Homepage: https://github.com/sdv-dev/SDGym

# Overview

Synthetic Data Gym (SDGym) is a framework to benchmark the performance of synthetic data generators
for non-temporal tabular data. SDGym is based on the paper [Modeling Tabular data using Conditional
GAN](https://arxiv.org/abs/1907.00503), and the project is part of the [Data to AI
Laboratory](https://dai.lids.mit.edu/) at MIT.

The benchmarking of a synthesizer is a process in which different datasets are generated by your
synthesizer. Then, each couple of real and synthetic data is evaluated with multiple scores.

## What is a synthesizer function?

**SDGym** evaluates the performance of **Synthesizer functions**.

These are python functions that take as input a numpy matrix with real data and output another
numpy matrix with the same shape filled with synthesized data.

Also, alongside the real data, some additional variables informing about the column contents
will be passed.

The complete list of inputs is:

* `real_data`: a 2D `numpy.ndarray` with the real data the your synthesizer will attempt to imitate.
* `categorical_columns`: a `list` with the indexes of any columns that should be considered
  categorical independently on their type.
* `ordinal_columns`: a `list` with the indexes of any integer columns that should be treated as
  ordinal values.

And the output should be a single 2D `numpy.ndarray` with the exact same shape as the `real_data`
matrix.

```python
def my_synthesizer_function(
    real_data: numpy.ndarray,
    categorical_columns: list,
    ordinal_columns: list
) -> syntehtesized_data: numpy.ndarray
```

## Benchmark datasets

The datasets used for the SDGym benchmarking process are grouped in three families:

* Simulated data generated using Gaussian Mixtures
    * grid
    * gridr
    * ring
* Simulated data generated using Bayesian Networks
    * asia
    * alarm
    * child
    * insurance
* Real world datasets
    * adult
    * census
    * covtype
    * credit
    * intrusion
    * mnist12
    * mnist28
    * news

Further details about how these datasets were generated can be found in the paper, and the code
that generated the simulated ones can be found inside the `sdgym/utils` folder of this repository.

All the datasets can also be found for download inside the [sgdym S3 bucket](
http://sdgym.s3.amazonaws.com/index.html) in the form of an `.npz` numpy matrix archive and
a `.json` metadata file that contains information about the dataset structure and their columns.

In order to load these datasets in the same format as they will be passed to your synthesizer function
you can use the `sdgym.load_dataset` function passing the name of the dataset to load.

In this example, we will load the `adult` dataset:

```python3
from sdgym import load_dataset

data, categorical_columns, ordinal_columns = load_dataset('adult')
```

This will return a numpy matrix with the data that will be passed to your synthesizer function,
as well as the list of indexes for the categorical and ordinal columns.

## SDGym Synthesizers

A part from the benchmark functionality, SDGym implements a collection of Synthesizers which are
either custom demo synthesizers or re-implementations of synthesizers that have been presented
in third party publications.

These Synthesizers are written as Python classes that can be imported from the `sdgym.synthesizers`
module and have the following methods:

* `fit`: Fits the synthesizer on the data. Expects the following arguments:
    * `data (numpy.ndarray)`: 2 dimensional Numpy matrix with the real data to learn from.
    * `categorical_columns (list or tuple)`: List of indexes of the columns that are categorical
      within the dataset.
    * `ordinal_columns (list or tuple)`: List of indexes of the columns that are ordinal within
      the dataset.
* `sample`: Generates new data resembling the original dataset. Expects the following arguments:
    * `n_samples (int)`: Number of samples to generate.
* `fit_sample`: Fits the synthesizer on the dataset and then samples as many rows as there were in
  the original dataset. It expects the same arguments as the `fit` method, and is ready to be
  directly passed to the `benchmark` function in order to evaluate the synthesizer performance.

This is the list of all the Synthesizers currently implemented, with references to the
corresponding publications when applicable.

|Name|Description|Reference|
|:--|:--|:--|
|`IdentitySynthesizer`|The synthetic data is the same as training data.||
|`UniformSynthesizer`|Each column in the synthetic data is sampled independently and uniformly.||
|`IndependentSynthesizer`|Each column in the synthetic data is sampled independently. Continuous columns are modeled by Gaussian mixture model. Discrete columns are sampled from the PMF of training data.||
|`CLBNSynthesizer`||[2]|
|`PrivBNSynthesizer`||[3]|
|`TableganSynthesizer`||[4]|
|`VEEGANSynthesizer`||[5]|
|`TVAESynthesizer`||[1]|
|`CTGANSynthesizer`||[1]|

## SDGym Leaderboard

This is the leaderboard with the scores that the SDGym Synthesizer obtained on the Benchmark
Datasets.

### Gaussian Mixture Simulated Data

|grid/syn_likelihood|grid/test_likelihood|gridr/syn_likelihood|gridr/test_likelihood|ring/syn_likelihood|ring/test_likelihood|
|-------------------|--------------------|--------------------|---------------------|-------------------|--------------------|
|              -3.06|               -3.06|               -3.06|                -3.07|              -1.7 |               -1.7 |
|              -3.68|               -8.62|               -3.76|               -11.6 |              -1.75|               -1.7 |
|              -4.33|              -21.67|               -3.98|               -13.88|              -1.82|               -1.71|
|             -10.04|              -62.93|               -9.45|               -72   |              -2.32|              -45.16|
|              -9.81|               -4.79|              -12.51|                -4.94|              -7.85|               -2.92|
|              -8.7 |               -4.99|               -9.64|                -4.7 |              -6.38|               -2.66|
|              -2.86|              -11.26|               -3.41|                -3.2 |              -1.68|               -1.79|
|              -5.63|               -3.69|               -8.11|                -4.31|              -3.43|               -2.19|

### Bayesian Networks Simulated Data

|asia/syn_likelihood|asia/test_likelihood|alarm/syn_likelihood|alarm/test_likelihood|child/syn_likelihood|child/test_likelihood|insurance/syn_likelihood|insurance/test_likelihood|
|-------------------|--------------------|--------------------|---------------------|--------------------|---------------------|------------------------|-------------------------|
|              -2.23|               -2.24|               -10.3|                -10.3|               -12  |                -12  |                   -12.8|                    -12.9|
|              -2.44|               -2.27|               -12.4|                -11.2|               -12.6|                -12.3|                   -15.2|                    -13.9|
|              -2.28|               -2.24|               -11.9|                -10.9|               -12.3|                -12.2|                   -14.7|                    -13.6|
|              -2.81|               -2.59|               -10.9|                -14.2|               -14.2|                -15.4|                   -16.4|                    -16.4|
|              -8.11|               -4.63|               -17.7|                -14.9|               -17.6|                -17.8|                   -18.2|                    -18.1|
|              -3.64|               -2.77|               -12.7|                -11.5|               -15  |                -13.3|                   -16  |                    -14.3|
|              -2.31|               -2.27|               -11.2|                -10.7|               -12.3|                -12.3|                   -14.7|                    -14.2|
|              -2.56|               -2.31|               -14.2|                -12.6|               -13.4|                -12.7|                   -16.5|                    -14.8|

### Real World Datasets

|adult/f1|census/f1|credit/f1|covtype/macro_f1|intrusion/macro_f1|mnist12/accuracy|mnist28/accuracy| news/r2|
|--------|---------|---------|----------------|------------------|----------------|----------------|--------|
|   0.669|    0.494|    0.72 |           0.652|             0.862|           0.886|           0.916| 0.14   |
|   0.334|    0.31 |    0.409|           0.319|             0.384|           0.741|           0.176|-6.28   |
|   0.414|    0.121|    0.185|           0.27 |             0.384|           0.117|           0.081|-4.49   |
|   0.375|    0    |    0    |           0.093|             0.299|           0.091|           0.104|-8.8    |
|   0.235|    0.094|    0    |           0.082|             0.261|           0.194|           0.136|-6.5e+06|
|   0.492|    0.358|    0.182|           0    |             0    |           0.1  |           0    |-3.09   |
|   0.626|    0.377|    0.098|           0.433|             0.511|           0.793|           0.794|-0.2    |
|   0.601|    0.391|    0.672|           0.324|             0.528|           0.394|           0.371|-0.43   |


# Install

## Requirements

**SDGym** has been developed and tested on [Python 3.5, and 3.6](https://www.python.org/downloads/)

Also, although it is not strictly required, the usage of a [virtualenv](https://virtualenv.pypa.io/en/latest/)
is highly recommended in order to avoid interfering with other software installed in the system
where **SDGym** is run.

## Install with pip

The easiest and recommended way to install **SDGym** is using [pip](https://pip.pypa.io/en/stable/):

```bash
pip install sdgym
```

This will pull and install the latest stable release from [PyPi](https://pypi.org/).

If you want to install it from source or contribute to the project please read the
[Contributing Guide](https://sdv-dev.github.io/SDGym/contributing.html#get-started) for
more details about how to do it.

## Compile C++ dependencies

Some of the third party synthesizers that SDGym offers, like the PrivBNSynthesizer, require
dependencies written in C++ need to be compiled.

In order to be able to use them, please do:

1. Clone or download the SDGym repository to your local machine:

```bash
git clone git@github.com:sdv-dev/SDGym.git
cd SDGym
```

2. make sure to have installed all the necessary dependencies to compile C++. In Linux
distributions based on Ubuntu, this can be done with the following command:

```bash
sudo apt-get install build-essential
```

3. Trigger the C++ compilation:

```bash
make compile
```

# Usage

## Benchmark

All you need to do in order to use the SDGym Benchmark, is to import and call the `sdgym.benchmark`
function passing it your synthesizer function:

```python3
from sdgym import benchmark

scores = benchmark(my_synthesizer_function)
```

The output of the `benchmark` function will be a `pd.DataFrame` containing the results obtained
by your synthesizer on each dataset, as well as the results obtained previously by the SDGym
synthesizers:

```
                        adult/accuracy  adult/f1  ...  ring/test_likelihood
IndependentSynthesizer         0.56530  0.134593  ...             -1.958888
UniformSynthesizer             0.39695  0.273753  ...             -2.519416
IdentitySynthesizer            0.82440  0.659250  ...             -1.705487
...                                ...       ...  ...                   ...
my_synthesizer_function        0.64865  0.210103  ...             -1.964966
```

## Using the SDGym Synthesizers

In order to use the synthesizer classes included in **SDGym**, you need to create an instance
of them, pass the real data to their `fit` method, and then generate new data using its
`sample` method.

Here's an example about how to use the `IdentitySynthesizer` to model and sample the `adult`
dataset.

```python3
from sdgym import load_dataset
from sdgym.synthesizers import IndependentSynthesizer

data, categorical_columns, ordinal_columns = load_dataset('adult')

synthesizer = IndependentSynthesizer()
synthesizer.fit(data, categorical_columns, ordinal_columns)

sampled = synthesizer.sample(3)
```

This will return a numpy matrix of sampeld data with the same columns as the original data and
as many rows as we have requested:

```
array([[5.1774925e+01, 0.0000000e+00, 5.3538445e+04, 6.0000000e+00,
        8.9999313e+00, 2.0000000e+00, 1.0000000e+00, 3.0000000e+00,
        2.0000000e+00, 1.0000000e+00, 3.7152294e-04, 1.9912617e-04,
        1.0767025e+01, 0.0000000e+00, 0.0000000e+00],
       [6.4843109e+01, 0.0000000e+00, 2.6462553e+05, 1.2000000e+01,
        8.9993210e+00, 1.0000000e+00, 0.0000000e+00, 1.0000000e+00,
        0.0000000e+00, 0.0000000e+00, 5.3685449e-06, 1.9797031e-03,
        2.2253288e+01, 0.0000000e+00, 0.0000000e+00],
       [6.5659584e+01, 5.0000000e+00, 3.6158912e+05, 8.0000000e+00,
        9.0010223e+00, 0.0000000e+00, 1.2000000e+01, 3.0000000e+00,
        0.0000000e+00, 0.0000000e+00, 1.0562389e-03, 0.0000000e+00,
        3.9998917e+01, 0.0000000e+00, 0.0000000e+00]], dtype=float32)
```

## Benchmarking the SDGym Synthesizers

If you want to re-evaluate the performance of any of the SDGym synthesizers, all you need to
do is to create an instance of the synthesizer and pass its `fit_sample` method to the
`benchmark` function.

```python3
from sdgym import benchmark
from sdgym.synthesizers import IndependentSynthesizer

synthesizer = IndependentSynthesizer()

leaderboard = benchmark(synthesizer.fit_sample)
```

The output will be a leaderboard like the one shown above.

# Related Projects

## SDV

[SDV](https://github.com/HDI-Project/SDV), for Synthetic Data Vault, is the end-user library for
synthesizing data in development under the [HDI Project](https://hdi-dai.lids.mit.edu/).
SDV allows you to easily model and sample relational datasets using Copulas thought a simple API.
Other features include anonymization of Personal Identifiable Information (PII) and preserving
relational integrity on sampled records.

## TGAN

[TGAN](https://github.com/sdv-dev/TGAN) is a GAN based model for synthesizing tabular data.
It's also developed by the [MIT's Data to AI Lab](https://dai-lab.github.io/) and is under
active development.


# References

[1] Lei Xu, Maria Skoularidou, Alfredo Cuesta-Infante, Kalyan Veeramachaneni. "Modeling tabular data using conditional gan." (2019) [(pdf)](https://papers.nips.cc/paper/8953-modeling-tabular-data-using-conditional-gan.pdf)

[2] C. Chow, Cong Liu. "Approximating discrete probability distributions with dependence trees." (1968) [(pdf)](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.133.9772&rep=rep1&type=pdf)

[3] Jun  Zhang, Graham Cormode, Cecilia M. Procopiuc, Divesh Srivastava, and Xiaokui Xiao. "Privbayes: Private data release via bayesian networks." (2017) [(pdf)](https://dl.acm.org/doi/pdf/10.1145/3134428)

[4] Noseong Park, Mahmoud Mohammadi, Kshitij Gorde, Sushil Jajodia, Hongkyu Park, Youngmin Kim. "Data synthesis based on generative adversarial networks." (2018) [(pdf)](https://dl.acm.org/ft_gateway.cfm?id=3242929&type=pdf)

[5] Akash Srivastava, Lazar Valkov, Chris Russell, Michael U. Gutmann, Charles Sutton. "Veegan: Reducing mode collapse in gans using implicit variational learning." (2017) [(pdf)](https://papers.nips.cc/paper/6923-veegan-reducing-mode-collapse-in-gans-using-implicit-variational-learning.pdf)
