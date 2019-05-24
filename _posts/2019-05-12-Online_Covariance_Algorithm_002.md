---
title: "Implementation of an Online Covariance Algorithm"
layout: post
thumb: "./img/ocov_add.png"
image: "./img/ocov_add.png"
excerpt: "This Python-class implements an online-algorithm for calculating a covariance-matrix."
tags: "online algorithm covariance merge vectorization"
---
**This Python-class implements an online-algorithm for calculating a covariance-matrix.**<!--more-->
The algorithm used can be found on [wikipedia](https://en.wikipedia.org/wiki/Algorithms_for_calculating_variance#Online). However, the Python-code given there is limited to two dimensions and uses for-loops.
This implementation is fully vectorized, which means that no for-loops or indices are involved when processing vectors and matrices. This should provide better performance and more versatility.

Not only the incremental growth of a covariance-matrix is implemented but also what the wikipedia article calls "combine".
Here I call the function ".merge()". It lends itself for a (probably faster) mini-batch implementation.


```python
import numpy as np
```


```python
class OnlineCovariance:
    """
    A class to calculate the mean and the covariance matrix
    of the incrementally added, n-dimensional data.
    """
    def __init__(self, order):
        """
        Parameters
        ----------
        order: int, The order (=="number of features") of the incrementally added
        dataset and of the resulting covariance matrix.
        """
        self._identity = np.identity(order)
        self._ones = np.ones(order)
        self._count = 0
        self._mean = np.zeros(order)
        self._cov = np.zeros((order,order))
        self._order = order
        
    @property
    def count(self):
        """
        int, The number of observations that has been added
        to this instance of OnlineCovariance.
        """
        return self._count 
   
    @property
    def mean(self):
        """
        double, The mean of the added data.
        """        
        return self._mean

    @property
    def cov(self):
        """
        array_like, The covariance matrix of the added data.
        """
        return self._cov

    @property
    def corrcoef(self):
        """
        array_like, The normalized covariance matrix of the added data.
        Consists of the Pearson Correlation Coefficients of the data's features.
        """
        if self._count < 1:
            return None
        variances = np.diagonal(self._cov)
        variance_rows = np.repeat(variances, self._order)
        variance_rows.shape = (self._order, self._order)
        return self._cov / np.sqrt(variance_rows * variance_rows.T)

    def add(self, observation):
        """
        Add the given observation to this object.
        
        Parameters
        ----------
        observation: array_like, The observation to add.
        """
        if self._order != len(observation):
            raise ValueError(f'Observation to add must be of size {self._order}')
            
        self._count += 1
        delta_at_nMin1 = np.array(observation - self._mean)
        self._mean += delta_at_nMin1 / self._count
        weighted_delta_at_n = np.array(observation - self._mean) / self._count
        D_at_n = np.repeat(weighted_delta_at_n, self._order)
        D_at_n.shape = (self._order, self._order)
        D = (delta_at_nMin1 * self._identity).dot(D_at_n.T)
        self._cov = self._cov * (self._count - 1) / self._count + D
    
    def merge(self, other):
        """
        Merges the current object and the given other object into a new OnlineCovariance object.

        Parameters
        ----------
        other: OnlineCovariance, The other OnlineCovariance to merge this object with.

        Returns
        -------
        OnlineCovariance
        """
        if other._order != self._order:
            raise ValueError(
                   f'''
                   Cannot merge two OnlineCovariances with different orders.
                   ({self._order} != {other._order})
                   ''')
            
        merged_cov = OnlineCovariance(self._order)
        merged_cov._count = self.count + other.count
        count_corr = (other.count * self.count) / merged_cov._count
        merged_cov._mean = (self.mean/other.count + other.mean/self.count) * count_corr
        mean_diffs = np.repeat(self._mean - other._mean, self._order)
        mean_diffs.shape = self._order, self._order
        merged_cov._cov = (self._cov * self.count \
                           + other._cov * other._count \
                           + mean_diffs * mean_diffs.T * count_corr) \
                          / merged_cov.count
        return merged_cov

```


```python
def create_correlated_dataset(n, mu, dependency, scale):
    latent = np.random.randn(n, dependency.shape[0])
    dependent = latent.dot(dependency)
    scaled = dependent * scale
    scaled_with_offset = scaled + mu
    return scaled_with_offset
```

### Demonstrate OnlineCovariance.add(observation)


```python
# Create an interdependent dataset
data = create_correlated_dataset( \
    10000, (2.2, 4.4, 1.5), np.array([[0.2, 0.5, 0.7],[0.3, 0.2, 0.2],[0.5,0.3,0.1]]), (1, 5, 3))
```


```python
# Calculate mean and covariance conventionally with numpy
conventional_mean = np.mean(data, axis=0)
conventional_cov = np.cov(data, rowvar=False)
conventional_corrcoef = np.corrcoef(data, rowvar=False)
```


```python
# Same calculations with OnlineCovariance
ocov = OnlineCovariance(data.shape[1])
for observation in data:
    ocov.add(observation)
```

### Assert that both ways yield the same result


```python
assert np.isclose(conventional_mean, ocov.mean).all(), \
"""
Mean should be the same with both approaches.
"""
assert np.isclose(conventional_cov, ocov.cov, atol=1e-3).all(), \
"""
Covariance-matrix should be the same with both approaches.
"""
assert np.isclose(conventional_corrcoef, ocov.corrcoef).all(), \
"""
Pearson-Correlationcoefficient-matrix should be the same with both approaches.
"""
```

None of the asserts fires. So this works.
Note the absolute tolerance-parameter ("atol=1e.-3") I had to put into the second assert.
It requires further investigation to see which approach is actually more exact and which one is faster.

### Demonstrate OnlineCovariance.merge()


```python
# create two differently correllated datasets
# (again, three dimensions)
data_part1 = create_correlated_dataset( \
    500, (2.2, 4.4, 1.5), np.array([[0.2, 0.5, 0.7],[0.3, 0.2, 0.2],[0.5,0.3,0.1]]), (1, 5, 3))
data_part2 = create_correlated_dataset( \
    1000, (5, 6, 2), np.array([[0.2, 0.5, 0.7],[0.3, 0.2, 0.2],[0.5,0.3,0.1]]), (1, 5, 3))
```


```python
ocov_part1 = OnlineCovariance(3)
ocov_part2 = OnlineCovariance(3)
ocov_both = OnlineCovariance(3)

# "grow" online-covariances for part 1 and 2 separately but also
# put all observations into the OnlineCovariance object for both.

for row in data_part1:
    ocov_part1.add(row)
    ocov_both.add(row)
    
for row in data_part2:
    ocov_part2.add(row)
    ocov_both.add(row)
    
ocov_merged = ocov_part1.merge(ocov_part2)

```

### Assert that count, mean and cov of both grown and merged objects are the same


```python
assert ocov_both.count == ocov_merged.count, \
"""
Count of ocov_both and ocov_merged should be the same.
"""
assert np.isclose(ocov_both.mean, ocov_merged.mean).all(), \
"""
Mean of ocov_both and ocov_merged should be the same.
"""
assert np.isclose(ocov_both.cov, ocov_merged.cov).all(), \
"""
Covarance-matrix of ocov_both and ocov_merged should be the same.
"""
assert np.isclose(ocov_both.corrcoef, ocov_merged.corrcoef).all(), \
"""
Pearson-Correlationcoefficient-matrix of ocov_both and ocov_merged should be the same.
"""
```
