# Bayesian networks in Python

This is an unambitious implementation of Bayesian networks in Python. For serious usage, you should probably be using an established projects, such as [pomegranate](https://pomegranate.readthedocs.io/en/latest/), [PyMC](https://docs.pymc.io/), [Stan](https://mc-stan.org/), [Edward](http://edwardlib.org/), and [Pyro](https://pyro.ai/).

## Installation

```sh
> pip install git+https://github.com/MaxHalford/rev --upgrade
```

## Usage

### Manual definition

As an example, let's use Judea Pearl's famous alarm network. The edges of the network can be provided manually when initialising a `BayesNet`:

```python
>>> import rev

>>> bn = rev.BayesNet(
...     ('Burglary', 'Alarm'),
...     ('Earthquake', 'Alarm'),
...     ('Alarm', 'John calls'),
...     ('Alarm', 'Mary calls')
... )

```

In Judea Pearl's example, the [conditional probability tables](https://www.wikiwand.com/en/Conditional_probability_table). We can set these manually by accessing the `cpts` attribute:

```python
>>> import pandas as pd

# P(Burglary)
>>> bn.cpts['Burglary'] = pd.Series({False: .999, True: .001})

# P(Earthquake)
>>> bn.cpts['Earthquake'] = pd.Series({False: .998, True: .002})

# P(Alarm | Burglary, Earthquake)
>>> bn.cpts['Alarm'] = pd.Series({
...     (True, True, True): .95,
...     (True, True, False): .05,
...
...     (True, False, True): .94,
...     (True, False, False): .06,
...
...     (False, True, True): .29,
...     (False, True, False): .71,
...
...     (False, False, True): .001,
...     (False, False, False): .999
... })

# P(John calls | Alarm)
>>> bn.cpts['John calls'] = pd.Series({
...     (True, True): .9,
...     (True, False): .1,
...     (False, True): .05,
...     (False, False): .95
... })

# P(Mary calls | Alarm)
>>> bn.cpts['Mary calls'] = pd.Series({
...     (True, True): .7,
...     (True, False): .3,
...     (False, True): .01,
...     (False, False): .99
... })

```

Once you've defined the network and specified the conditional probability tables, the `prepare` has to be called.

```python
>>> bn.prepare()

```

### Probabilistic inference

A Bayesian network is a generative model, and therefore can be used for many purposes. First of all, we can answer probabilistic queries, such as *what is the likelihood of there being a burglary if both John and Mary call?*. This can done via the `query` method, which returns the probability distribution of the possible outcomes.

```python
>>> bn.query('Burglary', event={'Mary calls': True, 'John calls': True})
Burglary
False    0.715828
True     0.284172
Name: P(Burglary), dtype: float64

```

By default, the answer is found via an exact inference method. For small networks this isn't very expensive to perform. However, for larger networks, you might want to prefer using [approximate inference](https://www.wikiwand.com/en/Approximate_inference). The latter is a class of methods that randomly sample the network and return an approximate answer. The latter is usually similar to the exact answer, provided enough iterations are performed. For instance, you can use [Gibbs sampling](https://www.wikiwand.com/en/Gibbs_sampling):

```python
>>> import numpy as np
>>> np.random.seed(42)

>>> bn.query('Burglary', event={'Mary calls': True, 'John calls': True}, algorithm='gibbs', n=1000)
Burglary
False    0.73
True     0.27
Name: P(Burglary), dtype: float64

```

### Missing value imputation

A usecase for probabilistic inference is to impute missing values. This `impute` method replaces the missing values with the most likely replacements given the present information. This is more sophisticated and possibly more accurate than simply replacing by the mean or the most common value. Additionally, such an approach can be much more efficient than [model-based iterative imputation](https://scikit-learn.org/stable/modules/generated/sklearn.impute.IterativeImputer.html#sklearn.impute.IterativeImputer).

```python
>>> from pprint import pprint

>>> sample = {
...     'Alarm': True,
...     'Burglary': True,
...     'Earthquake': False,
...     'John calls': None,  # missing
...     'Mary calls': None   # missing
... }

>>> sample = bn.impute(sample)
>>> pprint(sample)
{'Alarm': True,
 'Burglary': True,
 'Earthquake': False,
 'John calls': True,
 'Mary calls': True}

```

Note that the `impute` method can be seen as the equivalent of [`pomegranate`'s `predict` method](https://pomegranate.readthedocs.io/en/latest/BayesianNetwork.html#prediction).

### Random sampling

You can use a Bayesian network to generate random samples. The samples will follow the distribution induced by the network's structure and it's conditional probability tables.

```python
>>> pprint(bn.sample())
{'Alarm': False,
 'Burglary': False,
 'Earthquake': False,
 'John calls': False,
 'Mary calls': False}

>>> bn.sample(12)
    Alarm  Burglary  Earthquake  John calls  Mary calls
0   False     False       False       False       False
1   False     False       False       False       False
2   False     False       False       False       False
3   False     False       False        True       False
4   False     False       False       False       False
5   False     False       False       False       False
6   False     False       False       False       False
7   False     False       False       False       False
8   False     False       False       False       False
9   False     False       False       False       False
10  False     False       False       False       False
11  False     False       False       False       False

```

### Structure learning

On the way.

## Development

```sh
> git clone https://github.com/creme-ml/chantilly
> cd chantilly
> pip install -e ".[dev]"
> python setup.py develop
```
