# boosting

Build weak learners sequentially in a way that every next model takes in some way the results of the old model. There are two main types:

- weight-based boosting: examples where the old model did badly are given larger weights
- residual-based boosting: new model predicts not the labels but the residuals of the old model. For example, if sample 1 has target=0 and the model predicted 0.2, then the residual is -0.2
  - very successful type of algorithm (xgboost and friends)

Examples: (residual-based) lightgbm, xgboost, catboost... and weight-based: adaboost

## catboost

- automatically deals with categorical data (h2o GBM does that too). Uses one-hot encoding and has a "max" parameter
- has mean encoding built-in
- deals with interactions built-in
- creates symmetric trees which are less prone to overfitting and change less with different hyperparameters
- fast predictions
- has tricks that reduce overfitting (didn't go into detail)
- supports custom loss functions

# stacking

Simplest example: you split your training set in two parts. You train two models on the first part and predict on the second. Then you use the predictions on the second to train a new (meta) model. This is your final model. Meta model is usually a simpler model (linear model, lower depth random forest or a decision tree).

The idea is that

- the meta model will learn in which cases base models work better and will learn to weight them accordingly
- you also get lower variance since you average multiple models

Tips

- use diverse models as base learners: different algorithms, different feature engineering

StackNet combines models in a neural network. I didn't delve into this as I'm unlikely to use it in business.