# 00 Regression Package

## Table of Contents

1. Overview
    - What is a Lasso Regression?
    - Show Me the Papers 🤔
2. Set Up
    - Tools
    - Dependencies
3. Project Structure
4. Creating a Package
5. Testing
6. Building our Package
7. Publishing our Package
8. Serving our Model

## 1. Overview

In this section, we'll learn how to create a python package from a statistical model called, 
Lasso (or L1) Regression. The goal is not become an expert at implementing this method (I am 
confident you can master it on your own time), but rather to take seemingly challenging equation 
and bake into something others can use more than once. We'll finish the lesson by publishing 
our package, installing it, and serving a model with it.

By the end of this lesson, you will have the tools to create and publish your own Python packages.

## 1.1 What is a Lasso Regression?

$\text{minimize } J(\theta) = \frac{1}{2m} \sum_{i=1}^{m} (h_\theta(x^{(i)}) - y^{(i)})^2 + \lambda \sum_{j=1}^{n} |\theta_j|$


**Definition of Lasso Regression:**

Lasso Regression, or Least Absolute Shrinkage and Selection Operator, is a linear 
regression technique that introduces a penalty term to the traditional linear 
regression objective function. This penalty term is proportional to the absolute 
values of the regression coefficients. The goal of Lasso Regression is to not only 
minimize the sum of squared errors but also to minimize the sum of the absolute 
values of the coefficients, encouraging sparsity in the model. The strength of the 
penalty is controlled by a hyperparameter, often denoted as alpha.

**When to Use Lasso Regression:**

Lasso Regression is particularly useful in situations where feature selection is 
important. When dealing with datasets containing a large number of features, some of 
which may be irrelevant or redundant, Lasso Regression helps by driving the coefficients 
of less informative features to exactly zero. This inherent feature selection property 
makes Lasso Regression valuable in scenarios where interpretability and a sparse model are crucial.

For example, in genomics, where datasets might have thousands of genes, but only a few 
are expected to be relevant to a particular outcome, Lasso Regression can be employed to 
identify the subset of genes that play a significant role in the prediction.

**When Not to Use Lasso Regression:**

Lasso Regression may not be the best choice when all features in the dataset are genuinely 
informative and none should be completely eliminated. If it is important to retain all 
features without imposing sparsity, Ridge Regression, which introduces a penalty based on 
the square of the coefficients, might be a better alternative.

In cases where the number of observations is significantly smaller than the number of 
features (high-dimensional data) and there is multicollinearity among the features, Lasso 
Regression might encounter challenges in selecting the most relevant features. In such 
scenarios, techniques like Ridge Regression or Elastic Net Regression, which combines L1 
and L2 penalties, might be more suitable.

**Analogy:**

Think of Lasso Regression as a sculptor chiseling away excess material from a block of 
stone to reveal a refined and elegant statue. The sculptor (Lasso) carefully considers 
each part of the block (features) and decides whether it contributes meaningfully to the 
final artwork. If a part is deemed irrelevant, the sculptor chips it away, leaving only 
the essential components.

In this analogy, the block of stone represents the dataset, and the sculptor's decisions 
mirror the impact of the Lasso penalty on the regression coefficients. The result is a 
streamlined and sparse model, capturing only the essential features needed for accurate predictions.

## 1.2 Show Me the Papers 🤔


```python
from IPython.display import IFrame
```


```python
IFrame(src='https://arxivxplorer.com/', width=900, height=600)
```





<iframe
    width="900"
    height="600"
    src="https://arxivxplorer.com/"
    frameborder="0"
    allowfullscreen

></iframe>




## 2. Set Up

**Tools We'll be Using**

- [`numpy`](https://numpy.org/doc/stable/) -> "It is a Python library that provides 
a multidimensional array object, various derived objects (such as masked arrays 
and matrices), and an assortment of routines for fast operations on arrays, including 
mathematical, logical, shape manipulation, sorting, selecting, I/O, discrete Fourier 
transforms, basic linear algebra, basic statistical operations, random simulation and much more."
- [`setuptools`](https://setuptools.pypa.io/en/latest/) -> "Setuptools is a fully-featured, 
actively-maintained, and stable library designed to facilitate packaging Python projects."
- [`build`](https://pypa-build.readthedocs.io/en/stable/) -> "build manages 
pyproject.toml-based builds, invoking build-backend hooks as appropriate to build a 
distribution package. It is a simple build tool and does not perform any dependency management."
- [`twine`](https://twine.readthedocs.io/en/latest/) -> "Twine is a utility for publishing 
Python packages to PyPI and other repositories. It provides build system independent 
uploads of source and binary distribution artifacts for both new and existing projects."
- [`pytest`](https://docs.pytest.org/en/7.4.x/) -> "The pytest framework makes it 
easy to write small, readable tests, and can scale to support complex functional 
testing for applications and libraries."
- [`mlserver`](https://mlserver.readthedocs.io/en/latest/) -> "MLServer aims to 
provide an easy way to start serving your machine learning models through a REST 
and gRPC interface, fully compliant with KServe’s V2 Dataplane spec. Watch a quick 
video introducing the project here."


First open up your terminal and type the following. We'll need an virtual environment 
for all of our dependencies.

```sh
# with mamba or conda
mamba create -n lasso_dev python=3.11
mamba activate lasso_dev

# with virtualenv
python -m venv venv
## for linux and mac users
source venv/bin/activate
## for windows users
.\venv\Scripts\activate
```

Next, we'll install a few dependencies we'll need.

```sh
pip install pandas scikit-learn numpy build twine
```

## 3. Project Structure

By the end of the tutorial, our project will look as follows.

```md
.
├── build
│   ├── bdist.linux-x86_64
│   └── lib
│       └── lassoreg
│           ├── __init__.py
│           ├── __pycache__
│           │   └── regression.cpython-311.pyc
│           └── regression.py
├── dist
│   ├── lassoreg-0.1.0-py3-none-any.whl
│   └── lassoreg-0.1.0.tar.gz
├── lassoreg
│   ├── __init__.py
│   ├── __pycache__
│   │   └── regression.cpython-311.pyc
│   └── regression.py
├── lassoreg.egg-info
│   ├── dependency_links.txt
│   ├── PKG-INFO
│   ├── requires.txt
│   ├── SOURCES.txt
│   └── top_level.txt
├── pyproject.toml
├── README.md
└── tests
    └── test_lasso.py
```

Let's start by creating a directory for our project and package, plus a few other files we'll need.


```sh
mkdir ../first_package ../first_package/lassoreg ../first_package/tests
touch ../first_package/README.md ../first_package/pyproject.toml
```

Let's now get started building our package. 😎

## 4. Creating a Package

Let's start by loading some data and going through how lasso regression works using scikit-learn.


```python
from sklearn.linear_model import Lasso
from sklearn import datasets
import numpy as np
```


```python
datasets.load_diabetes(as_frame=True)['data'].head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>age</th>
      <th>sex</th>
      <th>bmi</th>
      <th>bp</th>
      <th>s1</th>
      <th>s2</th>
      <th>s3</th>
      <th>s4</th>
      <th>s5</th>
      <th>s6</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.038076</td>
      <td>0.050680</td>
      <td>0.061696</td>
      <td>0.021872</td>
      <td>-0.044223</td>
      <td>-0.034821</td>
      <td>-0.043401</td>
      <td>-0.002592</td>
      <td>0.019907</td>
      <td>-0.017646</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-0.001882</td>
      <td>-0.044642</td>
      <td>-0.051474</td>
      <td>-0.026328</td>
      <td>-0.008449</td>
      <td>-0.019163</td>
      <td>0.074412</td>
      <td>-0.039493</td>
      <td>-0.068332</td>
      <td>-0.092204</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.085299</td>
      <td>0.050680</td>
      <td>0.044451</td>
      <td>-0.005670</td>
      <td>-0.045599</td>
      <td>-0.034194</td>
      <td>-0.032356</td>
      <td>-0.002592</td>
      <td>0.002861</td>
      <td>-0.025930</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-0.089063</td>
      <td>-0.044642</td>
      <td>-0.011595</td>
      <td>-0.036656</td>
      <td>0.012191</td>
      <td>0.024991</td>
      <td>-0.036038</td>
      <td>0.034309</td>
      <td>0.022688</td>
      <td>-0.009362</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.005383</td>
      <td>-0.044642</td>
      <td>-0.036385</td>
      <td>0.021872</td>
      <td>0.003935</td>
      <td>0.015596</td>
      <td>0.008142</td>
      <td>-0.002592</td>
      <td>-0.031988</td>
      <td>-0.046641</td>
    </tr>
  </tbody>
</table>
</div>




```python
datasets.load_diabetes(as_frame=True)['target'].head()
```




    0    151.0
    1     75.0
    2    141.0
    3    206.0
    4    135.0
    Name: target, dtype: float64




```python
X, y = datasets.load_diabetes(return_X_y=True)
type(X), y.shape
```




    (numpy.ndarray, (442,))




```python
X = X[:150]
y = y[:150]

lasso = Lasso(alpha=1.0, max_iter=1000, tol=1e-4)
lasso.fit(X, y)

print(lasso.coef_)
print(lasso.intercept_)
```

    [  0.          -0.         239.9258791    0.          -0.
      -0.          -0.           0.         373.07866685   0.        ]
    150.40736813384746


y = b + X3(239.9258) + X9(373.0786) + e

Now, let's create our own implementation.


```python
%%writefile ../first_package/lassoreg/regression.py

import numpy as np

class LassoRegression:
    def __init__(self, alpha=1.0, max_iter=1000, tol=1e-4):
        self.alpha = alpha  # Regularization strength
        self.max_iter = max_iter  # Maximum number of iterations for optimization
        self.tol = tol  # Tolerance to determine convergence
        self.weights = None  # Coefficients
```

    Overwriting ../first_package/lassoreg/regression.py



```python
class LassoRegression:
    def __init__(self, alpha=1.0, max_iter=1000, tol=1e-4):
        self.alpha = alpha  # Regularization strength
        self.max_iter = max_iter  # Maximum number of iterations for optimization
        self.tol = tol  # Tolerance to determine convergence
        self.weights = None
my_lr_class = LassoRegression()
```


```python
my_lr_class.tol = 0.0002
my_lr_class.tol
```




    0.0002




```python
%%writefile -a ../first_package/lassoreg/regression.py


    def fit(self, X, y):
        # Initialize coefficients with zeros
        self.weights = np.zeros(X.shape[1] + 1)
        X_augmented = np.column_stack([np.ones(X.shape[0]), X])
        cost, gradient = self._cost_and_gradient(X_augmented, y, self.weights)

        for iteration in range(self.max_iter):

            self.weights -= self.alpha * gradient
            new_cost, new_gradient = self._cost_and_gradient(X_augmented, y, self.weights)
            if np.abs(new_cost - cost) < self.tol:
                break
            cost, gradient = new_cost, new_gradient
```

    Appending to ../first_package/lassoreg/regression.py



```python
%%writefile -a ../first_package/lassoreg/regression.py

    def _cost_and_gradient(self, X, y, weights):
        n_samples   = X.shape[0]
        predictions = np.dot(X, weights)
        residuals   = predictions - y
        cost        = (1 / (2 * n_samples)) * np.sum(residuals**2)
        l1_term     = self.alpha * np.sum(np.abs(weights[1:]))
        total_cost  = cost + l1_term
        gradient    = (1 / n_samples) * np.dot(X.T, residuals) + self.alpha * np.sign(weights)
        gradient[0] -= self.alpha * np.sign(weights[0])  # Exclude the intercept term
        return total_cost, gradient
```

    Appending to ../first_package/lassoreg/regression.py



```python
%%writefile -a ../first_package/lassoreg/regression.py

    def predict(self, X):
        X_augmented = np.column_stack([np.ones(X.shape[0]), X])
        return np.dot(X_augmented, self.weights)
```

    Appending to ../first_package/lassoreg/regression.py



```python
class LassoRegression:
    def __init__(self, alpha=1.0, max_iter=1000, tol=1e-4):
        self.alpha = alpha  # Regularization strength
        self.max_iter = max_iter  # Maximum number of iterations for optimization
        self.tol = tol  # Tolerance to determine convergence
        self.weights = None  # Coefficients

    def fit(self, X, y):
        # Initialize coefficients with zeros
        self.weights = np.zeros(X.shape[1] + 1)
        X_augmented = np.column_stack([np.ones(X.shape[0]), X])
        cost, gradient = self._cost_and_gradient(X_augmented, y, self.weights)

        for iteration in range(self.max_iter):

            self.weights -= self.alpha * gradient
            new_cost, new_gradient = self._cost_and_gradient(X_augmented, y, self.weights)
            if np.abs(new_cost - cost) < self.tol:
                break
            cost, gradient = new_cost, new_gradient

    def _cost_and_gradient(self, X, y, weights):
        n_samples   = X.shape[0]
        predictions = np.dot(X, weights)
        residuals   = predictions - y
        cost        = (1 / (2 * n_samples)) * np.sum(residuals**2)
        l1_term     = self.alpha * np.sum(np.abs(weights[1:]))
        total_cost  = cost + l1_term
        gradient    = (1 / n_samples) * np.dot(X.T, residuals) + self.alpha * np.sign(weights)
        gradient[0] -= self.alpha * np.sign(weights[0])  # Exclude the intercept term
        return total_cost, gradient

    def predict(self, X):
        X_augmented = np.column_stack([np.ones(X.shape[0]), X])
        return np.dot(X_augmented, self.weights)
```


```python
lasso2 = LassoRegression(alpha=1.0, max_iter=1000, tol=1e-4)
lasso2.fit(X, y)
```


```python
lasso2.weights
```




    array([ 1.49094905e+02, -6.15092379e-01,  8.50027238e-02,  1.20281928e+02,
            3.79675739e+01,  5.81608445e-01, -6.69098097e-01, -3.02614467e+01,
            2.75427723e+00,  1.41256309e+02,  1.26882417e+01])




```python
y_pred = lasso.predict(X)
y_pred2 = lasso2.predict(X)
y_pred[:20], y_pred2[:20]
```




    (array([172.63694312, 112.56436625, 162.13985803, 156.08973773,
            129.74383298, 125.28140937, 115.61884338, 136.59052179,
            159.6287421 , 185.05063898, 106.82661307, 118.62966678,
            132.01651185, 164.27673474, 132.32978307, 159.52719536,
            180.05860468, 163.52345725, 141.12618691, 142.73726374]),
     array([161.21914326, 128.72643819, 155.22165561, 150.62089449,
            140.16989569, 135.02954288, 133.33375212, 145.67818396,
            153.85120261, 162.73405856, 130.26789789, 137.6987127 ,
            138.91832348, 154.51632866, 137.45528102, 156.04587092,
            161.46642609, 157.78568804, 144.52716638, 142.21212783]))




```python
np.abs(y_pred - y_pred2).mean()
```




    9.963225305873683



## 5. Testing

Testing is one of the most important pieces of building good software so let's add a few tests using 
`hypohesis` and `pytest`.


```python
%%writefile ../first_package/tests/test_lasso.py

import pytest
import numpy as np
from lassoreg.regression import LassoRegression
from hypothesis import given, strategies as st

# Example strategy for generating random data
@st.composite
def generate_random_data(draw):
    n_samples = draw(st.integers(min_value=1, max_value=100))
    n_features = draw(st.integers(min_value=1, max_value=10))
    X = draw(st.lists(st.lists(st.floats(), min_size=n_features, max_size=n_features), min_size=n_samples, max_size=n_samples))
    y = draw(st.lists(st.floats(), min_size=n_samples, max_size=n_samples))
    return np.array(X), np.array(y)
```

    Overwriting ../first_package/tests/test_lasso.py


The following test checks if the LassoRegression model can fit to synthetic data. It ensures 
that the weights are updated after calling the fit method.


```python
%%writefile -a ../first_package/tests/test_lasso.py


def test_lasso_regression_fit():
    # Test if the LassoRegression model can fit to synthetic data
    X_train = np.array([[1, 2], [3, 4]])
    y_train = np.array([5, 6])
    model = LassoRegression(alpha=0.01, max_iter=1000, tol=1e-4)
    model.fit(X_train, y_train)
    assert model.weights is not None
```

    Appending to ../first_package/tests/test_lasso.py


This hypothesis test checks if the LassoRegression model converges with random 
data. It uses the `generate_random_data` strategy to generate random input data.


```python
%%writefile -a ../first_package/tests/test_lasso.py

@given(generate_random_data())
def test_lasso_regression_convergence(random_data):
    # Test if the LassoRegression model converges with random data
    X, y = random_data
    model = LassoRegression(alpha=0.01, max_iter=1000, tol=1e-4)
    model.fit(X, y)
    assert model.weights is not None
```

    Appending to ../first_package/tests/test_lasso.py


Our last test checks if the `LassoRegression` model can make predictions. It sets sample 
weights for the model and asserts that predictions are not None.


```python
%%writefile -a ../first_package/tests/test_lasso.py

def test_lasso_regression_predict():
    # Test if the LassoRegression model can make predictions
    X_test = np.array([[1, 2]])
    model = LassoRegression(alpha=0.01, max_iter=1000, tol=1e-4)
    model.weights = np.array([0.5, 0.2, 0.3])  # Sample weights for testing
    predictions = model.predict(X_test)
    assert predictions is not None
```

    Appending to ../first_package/tests/test_lasso.py


## 6. Building our Package

To build our package we'll need a `pyproject.toml` file. This specification is

> "TOML, or Tom's Obvious Minimal Language, is a data serialization language designed 
for configuration files. TOML files use a simple and readable syntax, making them easy 
for humans to write and understand. TOML is often employed for configuration purposes 
in software projects, providing a structured and organized way to specify settings and 
parameters. It uses key-value pairs, arrays, and tables to represent data hierarchies, 
and its minimalistic design aims to be clear and expressive while avoiding unnecessary 
complexity. TOML files are commonly used in various applications, including project 
configuration files, package metadata, and other settings where a straightforward and 
human-readable data format is desired.

The minimum configuration we'll need goes as follows.

```toml
[build-system]
requires = ["setuptools >= 65", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "lassoreg"
version = "0.1.0"
dependencies = ["numpy"]
```

The `build-system` specifies the libraries needed to build our packages with all of its 
dependencies and configurations. The `requires` focuses on specific libraries for the build 
process and `build-backend` specifies the tool that will turn the package into binaries.

the `project` section can get quite extensive but we'll leave it as is for now.


```python
%%writefile ../first_package/pyproject.toml

[build-system]
requires = ["setuptools >= 65", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "lassoreg_rap"
version = "0.1.0"
dependencies = ["numpy"]
```

    Overwriting ../first_package/pyproject.toml


The next thing we need to do is to install our package, and we'll do so with the following 
command from within the `first_package` directory.

```sh
python -m pip install -e .
```
The `-m` is used to pick up a module that is already available in our environment and the `-e` 
tells python that we will be _editing_ the package as we go.

Now the package is in our environment and we can test it. Open up a python or ipython session 
in your terminal and run the following.

```python
from lassoreg.regression import LassoRegression
```

Now that we know our package is working correctly, let's add a few more things to our `pyproject.toml`.


```python
%%writefile ../first_package/pyproject.toml
[build-system]
requires = ["setuptools >= 65", "setuptools_scm[toml]", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "lassoreg"
authors = [{name = "Ramon Perez", email = "ramon.perez@seldon.io"}]
description = "My wonderful Lasso Regression Python package"
version = "0.1.0"
readme = "README.md"
license = {text = "MIT License"}
requires-python = ">=3.11"
dependencies = [
    "numpy",
    "importlib_metadata"
]
keywords = [
    "statistics",
    "lasso",
    "lasso regression",
    "Regression",
    "Model",
    "Statistical Model"
]

[project.urls]
Source = "https://github.com/ramonpzg/architecting_tools/tree/main/first_package"
```

    Overwriting ../first_package/pyproject.toml


A quick word on versioning. Most Python packages follow the MAJOR.MINOR.PATCH and it would be useful to 
get familiarized with it. Please note, there are a few different conventions for numbering as well.

- Major: A completely new version of the package where breaking changes are expected.
- Minor: New features backwards compatible.
- Patch: Bug fixes and improvements to code quality.

## 7. Publishing our Package

Before we publish it, let's first populate the README of our project. We'll use the following paragraphs.


```python
%%writefile ../first_package/README.md

# Lasso Regression Package

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)

## Overview

This Python package provides a simple implementation of Lasso Regression (L1 regularization) 
using the Python Standard Library and `NumPy`. Lasso Regression is a linear regression 
technique that adds a penalty term proportional to the absolute values of the regression 
coefficients, promoting sparsity in the model.

## Installation

```bash
pip install lassoreg
```

## Usage

```python
from lassoreg.regression import LassoRegression

# Create an instance of Lasso Regression
lasso_model = LassoRegression(alpha=0.01, max_iter=1000, tol=1e-4)

# Fit the model to training data
lasso_model.fit(X_train, y_train)

# Make predictions on new data
predictions = lasso_model.predict(X_test)
```

## Documentation

For detailed information on the parameters and methods, please refer to the docstring in the source code.

## Example

An example of generating synthetic data and fitting the Lasso Regression model is provided in the `example` directory.

```bash
cd example
python example.py
```

## Testing

To run the unit tests, use the following command:

```bash
pytest tests
```

## License

This package is licensed under the [MIT License](LICENSE).
```

    Overwriting ../first_package/README.md


Next, we need to install `build` and `twine` to create our package and publish it.

```sh
pip install build twine
```

Let's first build our package form the main directory of our package.

```sh
python -m build
```

You'll see a new directory called `dist` and in it, you'll see two files, a `.whl` and a `.tar.gz`. Here 
is some additional information about the two.

1. Wheel (`.whl`) Package:
    - Purpose: The Wheel format is a binary distribution format that aims to be more efficient 
    for installing Python packages compared to the traditional source distribution formats like .tar.gz.
    - Advantages:
        - Faster installation: Wheels are pre-compiled, making installation faster compared to source distributions.
        - Simplified package management: Wheels include metadata and dependencies, streamlining the installation process.
        - Platform-specific: Wheels can be platform-specific, optimizing compatibility with different systems.
    - Use Case: Wheels are commonly used for distributing and installing Python packages, especially for projects with binary extensions or dependencies.
2. Source Tarball (`.tar.gz`) Package:
    - Purpose: The source tarball is a compressed archive of the project's source code and related files. It contains everything needed to build and install the project.
    - Advantages:
        - Portability: Source tarballs can be used on any platform, as they provide the project's source code.
        - Customization: Users can modify the source code before building and installing the package.
    - Use Case: Source tarballs are often used when distributing open-source projects, allowing users to build and install the software on their system.

### Publishing to GitHub

The nice thing about `pip` is that it allows us to install a project directly from a repo or 
and `tarball`. While either of these two options can be quite slow due to the whole repo 
being contained in the download, they can still be quite useful for working with packages 
that haven't been published or features in different branches. Let's walk over the options.

Note: These examples are borrowed from the excellent tutorial on packaging by 
[The Carpentries](https://carpentries.org), and you can find it 
[here](https://carpentries-incubator.github.io/python_packaging/instructor/05-publishing.html).

```sh
pip install "git+https://github.com/ramonpzg/architecing_tools"
pip install "git+https://github.com/ramonpzg/architecing_tools@1.2.3"
pip install "git+https://github.com/ramonpzg/architecing_tools@branch"
pip install "git+https://github.com/ramonpzg/architecing_tools@1a2b3c4"
pip install "https://github.com/ramonpzg/architecing_tools/archive/1.2.3.zip"
```

you can also add them all to your dependencies in your pyproject.toml

```py
dependencies = [
    "mypkg @ https://github.com/user/ramonpzg/architecing_tools/1.2.3.zip",
]
```

### Publishing to PyPi

You will first need to create an account at [pypi.org](https://pypi.org), authenticate your 
account

Go to the website.  
![pypi1](../images/pypi1.png)

Create an account.  
![pypi2](../images/pypi2.png)

Enable two-factor authentication (Optional but recommended).  
![pypi3](../images/pypi3.png)

Once you finish creating your account, you will need to create an API token by going 
to **manage** >> **account** and then to the following section.

![pypi4](../images/pypi4.png)

Once you have your API token, create a file called `.pypirc` in your home directory and then add 
the following lines.

```sh
[pypi]
  username = __token__
  password = your_api_token
```

Perfect, now we're ready to publish our package, but first, let's check that there are no issues 
with the wheels and the tarball we created with our package.

```sh
twine check dist/*
```

And, for the last step.

```sh
twine upload dist/*
```

![pypi5](../images/pypi5.png)

Excellent, we have successfully uploaded our package and can now install it via pip.

## 8. Serving our Model

Here are the steps we'll be taking in this section.

1. Download Library
2. Train Model
3. Save Pickle File
4. Serve it
5. Make Predictions


```python
!pip install lassoreg
```

    Requirement already satisfied: lassoreg in /home/ramonperez/mambaforge/envs/lasso_dev/lib/python3.11/site-packages (0.1.0)
    Requirement already satisfied: numpy in /home/ramonperez/mambaforge/envs/lasso_dev/lib/python3.11/site-packages (from lassoreg) (1.26.2)
    Requirement already satisfied: importlib-metadata in /home/ramonperez/mambaforge/envs/lasso_dev/lib/python3.11/site-packages (from lassoreg) (7.0.0)
    Requirement already satisfied: zipp>=0.5 in /home/ramonperez/mambaforge/envs/lasso_dev/lib/python3.11/site-packages (from importlib-metadata->lassoreg) (3.17.0)



```python
from lassoreg.regression import LassoRegression
```


```python
lasso3 = LassoRegression(alpha=1.0, max_iter=1000, tol=1e-4)
lasso3.fit(X, y)
```


```python
y
```




    array([151.,  75., 141., 206., 135.,  97., 138.,  63., 110., 310., 101.,
            69., 179., 185., 118., 171., 166., 144.,  97., 168.,  68.,  49.,
            68., 245., 184., 202., 137.,  85., 131., 283., 129.,  59., 341.,
            87.,  65., 102., 265., 276., 252.,  90., 100.,  55.,  61.,  92.,
           259.,  53., 190., 142.,  75., 142., 155., 225.,  59., 104., 182.,
           128.,  52.,  37., 170., 170.,  61., 144.,  52., 128.,  71., 163.,
           150.,  97., 160., 178.,  48., 270., 202., 111.,  85.,  42., 170.,
           200., 252., 113., 143.,  51.,  52., 210.,  65., 141.,  55., 134.,
            42., 111.,  98., 164.,  48.,  96.,  90., 162., 150., 279.,  92.,
            83., 128., 102., 302., 198.,  95.,  53., 134., 144., 232.,  81.,
           104.,  59., 246., 297., 258., 229., 275., 281., 179., 200., 200.,
           173., 180.,  84., 121., 161.,  99., 109., 115., 268., 274., 158.,
           107.,  83., 103., 272.,  85., 280., 336., 281., 118., 317., 235.,
            60., 174., 259., 178., 128.,  96., 126.])




```python
import pickle
```


```python
!mkdir ../models
```


```python
with open('../models/model.pkl', 'wb') as f:
    pickle.dump(lasso3, f)
```


```python
lasso3 = pickle.load(open('../models/model.pkl', 'rb'))
```


```python
lasso3
```




    <lassoreg.regression.LassoRegression at 0x7f2f2800f3d0>




```python
X[0, None].shape, X[0, None]
```




    ((1, 10),
     array([[ 0.03807591,  0.05068012,  0.06169621,  0.02187239, -0.0442235 ,
             -0.03482076, -0.04340085, -0.00259226,  0.01990749, -0.01764613]]))




```python
lasso3.predict(X[0, None])
```




    array([161.21914326])



You can also do it in one line.


```python
lasso_model = pickle.load(open('../server/models/model.pkl', 'rb'))
```


```python
%%writefile ../server/my_lasso.py

from mlserver.codecs import decode_args
from mlserver import MLModel
from lassoreg.regression import LassoRegression
from sklearn import datasets
import numpy as np
import os

class LassoCLS(MLModel):
    async def load(self) -> bool:
        X, y = datasets.load_diabetes(return_X_y=True)
        X = X[:150]
        y = y[:150]
        self.model = LassoRegression(alpha=1.0, max_iter=1000, tol=1e-4)
        self.model.fit(X, y)
        return True

    @decode_args
    async def predict(self, features: np.ndarray) -> np.ndarray:
        return self.model.predict(features)
```

    Writing ../server/my_lasso.py



```python
%%writefile ../server/model-settings.json
{
    "name": "lasso_service",
    "implementation": "my_lasso.LassoCLS"
}
```

    Writing ../server/model-settings.json



```python
%%writefile ../server/settings.json
{
    "http_port": 7090,
    "grpc_port": 7050,
    "metrics_port": 9090
}
```

    Writing ../server/settings.json


Now that we have the files needed for our service, we can run the following line to start our server.
```sh
mlserver start name_of_directory
```


```python
from mlserver.codecs import NumpyCodec
import requests
```


```python
endpoint = "http://localhost:7090/v2/models/lasso_service/infer"
```


```python
x_0 = X[0, None]
x_0
```




    array([[ 0.03807591,  0.05068012,  0.06169621,  0.02187239, -0.0442235 ,
            -0.03482076, -0.04340085, -0.00259226,  0.01990749, -0.01764613]])




```python
inference_request = {
    'inputs': [
        NumpyCodec.encode_input(name="features", payload=x_0).dict()
    ]
}
inference_request
```




    {'inputs': [{'name': 'features',
       'shape': [1, 10],
       'datatype': 'FP64',
       'parameters': {'content_type': 'np'},
       'data': [0.038075906433423026,
        0.05068011873981862,
        0.061696206518683294,
        0.0218723855140367,
        -0.04422349842444599,
        -0.03482076283769895,
        -0.04340084565202491,
        -0.002592261998183278,
        0.019907486170462722,
        -0.01764612515980379]}]}




```python
response = requests.post(endpoint, json=inference_request)
print(response)
print(response.json())
```

    <Response [200]>
    {'model_name': 'lasso_service', 'id': 'aeaba036-7b74-42df-b6d1-04a87d21822a', 'parameters': {}, 'outputs': [{'name': 'output-0', 'shape': [1, 1], 'datatype': 'FP64', 'parameters': {'content_type': 'np'}, 'data': [161.2191432587962]}]}



```python

```
