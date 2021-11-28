# general

**No free lunch theorem** says that there is no method that outperforms all others on all tasks. Or put differently, you can always find a task where a method performs poorly compared to others. Reason is that each method makes assumptions about the data that could be wrong.

# linear models

They split space into 2 subspaces (for classification)

Good for sparse, high dimensional data

**examples:** logistic regression, support vector machines, 

**implementations:** Scikit-Learn, Vowpal Wabbit

# tree-based methods

Split space into "boxes", then use constant predictions within boxes

Good default method for tabular data, but hard to capture linear dependencies

**examples:** random forest, gradient boosted machines

**implementations:** LightGBM, XGboost, Scikit-Learn for random forest

# kNN-based

Find N similar objects and use their average as prediction

Good for generating features, not that often used for generating predictions directly

# neural networks

**implementations:** Tensorflow, Keras, Pytorch, MxNet, Lasagne