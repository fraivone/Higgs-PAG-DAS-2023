## Summary
This repo contains code to finalize the Higgs PAG exercise from https://indico.desy.de/event/38207/contributions/152491/.
A general intro and analysis cuts was provided by Matteo [link](https://indico.desy.de/event/38207/contributions/152491/attachments/86014/114459/141023_PODASExercise_Matteo.pdf).
I prefer an interactive environment for the more attempt-intense part of the exercise. Therefore I decided to split the analysis into 2 parts:
* Event calibration and selection via  [Column Flow](https://github.com/columnflow/columnflow) on a DESY node
* Candidate reconstruction locally via plain awkward arrays on a jupyter notebook.


I have used the `parquet` files naturally produced by ColumnFlow to move the selected events from DESY to my local machine.

## Why 2 Jupyter notebooks?
Columnar analysis imposes a different style of handling the events and forces the collections to be immutable. 
These differences with respect to classic one-event-at-a-time style emerge when trying to form all possible Z boson candidates from a list of leptons.
The old-fashioned for-loop wouldn't satisfy the columnar approach. The way to go is using`awkward` `cartesian` product.
However the product allows the same element to form more than one pair when the number of array has `length >= 2`.
Given 3posimuons and 2negamuons 6 pairs are possible. However no more than 2 *real* Z candidates exist, as each Z candidate decays in a posi-nega pair. Enforcing this logic isn't straightforward with awkward arrays:
```python
>>> print(one),print(two)
[['mu1+', 'mu2+', 'mu3+']]
[['mu1-', 'mu2-']]
>>> cart = ak.cartesian([one,two])
>>> for k in range(len(cart[0,:])):
...     print(cart[0,k])
('mu1+', 'mu1-')
('mu1+', 'mu2-')
('mu2+', 'mu1-')
('mu2+', 'mu2-')
('mu3+', 'mu1-')
('mu3+', 'mu2-')
```
Now if these were real muon objects, the pairs would be Z candidates. However, when selecting one pair, we should also discard all other pairs that share elements with it. For example selecting the first pair `('mu1+', 'mu1-')` as the best Z candidates, one must also discard 
```python
('mu1+', 'mu2-')
('mu2+', 'mu1-')
('mu3+', 'mu1-')
```
as they share elements with it and thus remaining with only 2 possible, mutually exclusive, candidates.
```python
('mu2+', 'mu2-')
('mu3+', 'mu2-')
```

---
I came up with 2 implementations of the same code:
* `HiggsAnalysis_withDuplicates` which allows combinatorics repetitions of objects in many pairs 
* `HiggsAnalysis_DuplicateMuonSuppression`which doesn't 
Both implement the same exact analysis differing in the way leptons are paired. 

## env
``` python
Python 3.8.10
hist 2.6.2
numpy 1.22.1
matplotlib 3.5.2
```