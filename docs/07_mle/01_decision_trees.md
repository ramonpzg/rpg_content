# 01 Packaging a Decision Tree Classifier

## Table of Content

1. Overview
2. Package
3. Instructions
4. Testing

## 1. Overview



## 2. Package

We will be using this package in a little bit.

```py
import numpy as np

class DecisionTreeClassifier:
    def __init__(self, max_depth=None):
        self.max_depth = max_depth
        self.tree = None

    def fit(self, X, y):
        self.tree = self._build_tree(X, y, depth=0)

    def _build_tree(self, X, y, depth):
        unique_classes = np.unique(y)

        # Base cases
        if len(unique_classes) == 1:
            # If all labels are the same, create a leaf node
            return {'class': unique_classes[0]}

        if self.max_depth is not None and depth == self.max_depth:
            # If max depth is reached, create a leaf node with the majority class
            majority_class = np.argmax(np.bincount(y))
            return {'class': majority_class}

        # Find the best split
        best_split = self._find_best_split(X, y)

        if best_split is None:
            # If no split improves purity, create a leaf node with the majority class
            majority_class = np.argmax(np.bincount(y))
            return {'class': majority_class}

        feature_index, threshold = best_split

        # Split the data
        left_indices = X[:, feature_index] <= threshold
        right_indices = ~left_indices

        # Recursively build left and right subtrees
        left_subtree = self._build_tree(X[left_indices], y[left_indices], depth + 1)
        right_subtree = self._build_tree(X[right_indices], y[right_indices], depth + 1)

        # Return a node representing the split
        return {'feature_index': feature_index, 'threshold': threshold,
                'left': left_subtree, 'right': right_subtree}

    def _find_best_split(self, X, y):
        num_features = X.shape[1]
        best_gini = float('inf')
        best_split = None

        for feature_index in range(num_features):
            feature_values = np.unique(X[:, feature_index])
            for value in feature_values:
                left_indices = X[:, feature_index] <= value
                right_indices = ~left_indices

                gini = self._calculate_gini_index(y[left_indices], y[right_indices])

                if gini < best_gini:
                    best_gini = gini
                    best_split = (feature_index, value)

        return best_split

    def _calculate_gini_index(self, left_labels, right_labels):
        total_size = len(left_labels) + len(right_labels)

        if total_size == 0:
            return 0

        p_left = len(left_labels) / total_size
        p_right = len(right_labels) / total_size

        gini_left = 1 - np.sum((np.bincount(left_labels) / len(left_labels))**2)
        gini_right = 1 - np.sum((np.bincount(right_labels) / len(right_labels))**2)

        gini_index = p_left * gini_left + p_right * gini_right

        return gini_index

    def predict(self, X):
        return np.array([self._predict_tree(self.tree, x) for x in X])

    def _predict_tree(self, node, x):
        if 'class' in node:
            # If it's a leaf node, return the predicted class
            return node['class']
        else:
            # Traverse the tree recursively
            if x[node['feature_index']] <= node['threshold']:
                return self._predict_tree(node['left'], x)
            else:
                return self._predict_tree(node['right'], x)
```

## 3. Instructions

- install pipx
- install pdm
- pdm
- mkdir second_package
- cd second_package
- pdm init
    - 0
    - y
    - y
    - decision_tree
    - default
    - description
    - pdm-backend
    - MIT
    - Author default
    - Email default
    - default
        * walk through the output
        - go to pyproject.toml and show that there are no dependencies
        - use pdm add for numpy
        - use pdm install
    - explain differences between using src and the name of the package
- Create decision tree file
- add decision tree step by step
- test scikit-learn implementation
- test our from-scratch implementation
- create unit tests
- add repo url to toml
- add to github with tag
- pdm build
- pdm publish --no-build
- go to pypi and look for package
- install it and use it to test it
    - create env
    - activate env
    - pip install package ipython scikit-learn
    - open ipython
        from sklearn.datasets import make_classification
        from decision_tree.classification import DecisionTreeClassifier
        X, y = make_classification(1000, 10, random_state=0)
        dt_classifier = DecisionTreeClassifier(max_depth=3)
        dt_classifier.fit(X, y)
        predictions = dt_classifier.predict(X)
        predictions
- add some more functionality
- install it in a new env 
- train a model
- load it and test it with MLServer

## 4. Testing




```python
%%writefile ../second_package/tests/test_decisions.py

import pytest
import numpy as np
from decision_tree.classification import DecisionTreeClassifier
from hypothesis import given, strategies as st

# Example strategy for generating random data
@st.composite
def generate_random_data(draw):
    n_samples = draw(st.integers(min_value=1, max_value=100))
    n_features = draw(st.integers(min_value=1, max_value=10))
    X = draw(st.lists(st.lists(st.floats(), min_size=n_features, max_size=n_features), min_size=n_samples, max_size=n_samples))
    y = draw(st.lists(st.integers(), min_size=n_samples, max_size=n_samples))
    return np.array(X), np.array(y)
```


```python
import pytest
import numpy as np
from decision_tree.classification import DecisionTreeClassifier
from hypothesis import given, strategies as st
```


```python
@st.composite
def generate_random_data(draw):
    n_samples = draw(st.integers(min_value=1, max_value=100))
    n_features = draw(st.integers(min_value=1, max_value=10))
    X = draw(st.lists(st.lists(st.floats(), min_size=n_features, max_size=n_features), min_size=n_samples, max_size=n_samples))
    y = draw(st.lists(st.integers(), min_size=n_samples, max_size=n_samples))
    return np.array(X), np.array(y)
```




```python
def test_decision_tree_classifier_fit():
    # Test if the DecisionTreeClassifier model can fit to synthetic data
    X_train = np.array([[1, 2], [3, 4]])
    y_train = np.array([0, 1])
    model = DecisionTreeClassifier(max_depth=3)
    model.fit(X_train, y_train)
    assert model.tree is not None
```


```python
test_decision_tree_classifier_fit()
```


```python
%%writefile -a ../second_package/tests/test_decisions.py

def test_decision_tree_classifier_fit():
    # Test if the DecisionTreeClassifier model can fit to synthetic data
    X_train = np.array([[1, 2], [3, 4]])
    y_train = np.array([0, 1])
    model = DecisionTreeClassifier(max_depth=3)
    model.fit(X_train, y_train)
    assert model.tree is not None
```




```python
@given(generate_random_data())
def test_decision_tree_classifier_predict(random_data):
    # Test if the DecisionTreeClassifier model can make predictions with random data
    X, y = random_data
    model = DecisionTreeClassifier(max_depth=3)
    model.fit(X, y)

    # Ensure predictions are valid
    predictions = model.predict(X)
    assert all(isinstance(pred, int) for pred in predictions)
```


```python
test_decision_tree_classifier_predict()
```


```python
%%writefile -a ../second_package/tests/test_decisions.py

@given(generate_random_data())
def test_decision_tree_classifier_predict(random_data):
    # Test if the DecisionTreeClassifier model can make predictions with random data
    X, y = random_data
    model = DecisionTreeClassifier(max_depth=3)
    model.fit(X, y)

    # Ensure predictions are valid
    predictions = model.predict(X)
    assert all(isinstance(pred, int) for pred in predictions)
```




```python
def test_decision_tree_classifier_max_depth():
    # Test if the DecisionTreeClassifier respects the max_depth parameter
    X_train = np.array([[1, 2], [3, 4]])
    y_train = np.array([0, 1])
    model = DecisionTreeClassifier(max_depth=1)
    model.fit(X_train, y_train)
    
    # Ensure the tree depth does not exceed the specified max_depth
    def get_tree_depth(node):
        if 'class' in node:
            return 1
        else:
            return 1 + max(get_tree_depth(node['left']), get_tree_depth(node['right']))
    
    assert get_tree_depth(model.tree) <= model.max_depth
```


```python
test_decision_tree_classifier_max_depth()
```


```python
%%writefile -a ../second_package/tests/test_decisions.py

def test_decision_tree_classifier_max_depth():
    # Test if the DecisionTreeClassifier respects the max_depth parameter
    X_train = np.array([[1, 2], [3, 4]])
    y_train = np.array([0, 1])
    model = DecisionTreeClassifier(max_depth=1)
    model.fit(X_train, y_train)
    
    # Ensure the tree depth does not exceed the specified max_depth
    def get_tree_depth(node):
        if 'class' in node:
            return 1
        else:
            return 1 + max(get_tree_depth(node['left']), get_tree_depth(node['right']))
    
    assert get_tree_depth(model.tree) <= model.max_depth
```

We can now run `pytest tests` to make sure that our code is solid.
