# minimizing the objective function
In machine learning the model is supposed to be a good fit to the data. A "good fit" means is that the deviation between the model and the data is low. 

You need a mathematical definition of this deviation, and this is called the *objective function*. The word "objective" here refers to the fact that it is the objective of the machine learning algorithm to optimize (i.e., minimize) this function to get the lowest possible deviation.

An objective function can contain other parts, not just the deviation / cost part. It may contain a complexity penalty as in ridge regression. Such objective function not only minimizes the deviation between the data and the model, but also at the same time minimizes the complexity of the model.

[Link: difference between loss, cost and objective functions](http://stats.stackexchange.com/questions/179026/objective-function-cost-function-loss-function-are-they-the-same-thing
)

## loss function
Loss function is a measure of the cost of making a wrong prediction. A number quantifying the difference ("putting a price on it") btw the true value and the predicted value. Calling it a cost instead of just a difference is intentional since you could use asymmetric loss functions, eg. underpredicting is more costly than overpredicting.

The formula below says that the loss function has two parameters: the true value of the outcome $y$ and the predicted value given some set of parameters $\hat w$.

$$
L(y, f_{\hat w}(x))
$$

Common loss functions include squared error and absolute error. The most common one is root means squared error (RMSE). Taking the root resets the units back from squared to original ones.

$$
RMSE = \sqrt {\dfrac {1} {N}\sum _{i=1}^{N}\left( y-\hat {y}\right) ^{2}}
$$

## training, generalization and test error

### training error
Training error is the average loss over the datapoints that you used to learn the parameters. This error is optimistic, ie too low. 

When adding new features the train error can never be higher than with less features since you can always set the weight for the new feature to zero. More generally,

> training error always gets lower with increasing model complexity

It is a bad indication of predictive performance outside the training dataset.

### true error
True error, or generalization error is the error on the population. It is the error over all possible predictor and outcome combinations, weighted by how likely the combinations are.

> True error is a theoretical idea, in practice you can only approximate it

The relation btw the true error and complexity of the model (trained on training data) depends on the underlying true model. Typically, though, there is a "sweet spot" of complexity for a given problem, and for a given training sample size.

### test error
Test error involves holding out some data and testing the model on it. It is a proxy for "everything else", an approximation of the true error.

> Test error is always calculated using the model trained on the training dataset. Also, any data transformations must match training data transformations.

**Overfitting** happens when you increase the complexity of the model and the training error gets lower, but the test error gets higher.

## sources of error

### irreducible error
Irreducible error is the variance of the difference btw the true relationship of X and y (the "perfect model" for this dataset), and the actual outcomes. This noise has zero mean. If not zero, you would place that value inside the true function.

It is irreducible since you cannot find a model that is better than the true relationship. This is the error inherent in the data, it is everything we did not capture in our predictor set but is relevant for the outcome.

$$
\epsilon_i = f_{w(true)}(X_i) + y_i
$$

Partly, this error is dependent on the inherent complexity of the outcome: how many predictors do you need to accurately estimate it? It will contain all the predictors you did not include in the model.

### bias
Bias is the extent to which the model you chose can fit the true relationship btw X and y, given your sample size. It is difference btw the true function and the average function for our model across an infinite number of datasets of sample size N.

Imagine fitting the model many times over datasets of size N. The average of these models is the best model that we can get given our restrictions (the imperfect model, the incomplete dataset). Bias is asking if the model that we chose is flexible enough to capture the true relationship. Or, another way, bias is the difference between our best model and the true model.

Formally, the average function is the expected function over all training datasets of size N. Here, and with variance below, we look at a specific target datapoint $X_t$.

$$
f_{\overline {w}}\left( X_t\right) =E_{train}\left[ f_{\hat {w}}\left( X_t\right) \right]
$$

And bias is the difference btw the true function and that average function. Both irreducible error and variance (described below) are squared differences, so we need to square bias to put them on equal terms.

$$
bias(f_{\hat {w}}(X_t)) = f_w(X_t) - f_{\overline {w}}(X_t)
$$

### variance
Variance tells you to what extent do models fitted on different datasets vary from one another. It is the difference btw models trained on individual datasets and the average model.

> Both bias and variance are theoretical. We don't have the true function and we don't have unlimited datasets of size N.

Formally, variance is the expected squared difference between the prediction of a model fit on a specific dataset and the average model. So, it is the average deviation between a specific fit and the expected fit.

$$
var(f_{\hat {w}}(X_t)) = E\left[(f_{\hat {w}}(X_t) - f_{\overline {w}}(X_t))^2\right]
$$

### relationship btw bias and variance
> More complex models have higher variance but lower bias, and vice versa

Bias and variance are combined into mean squared error. Since bias gets lower with model complexity, and variance gets higher, the goal is to find the sweet spot.

> the goal of machine learning is to find the optimal combination of bias and variance of a model, ie minimum prediction error

$$
MSE = bias^2 + variance \\\
meanError = irreducibleError + bias^2 + variance.
$$

### error against number of datapoints
The more datapoints we have, the better our model is in extrapolating to new datapoints. The true error gets lower. It approaches irreducible error + bias.

> this means that with more and more data, the limiting factor becomes the flexibility of the model. With a lot of data, complex models are more useful.

But with greater N, the training error gets higher since the datapoints are more variable. Train error also approaches irreducible error + bias, ie it approaches the true error, the error on the whole population.

### mathematically deriving sources of bias

![expected prediction error](https://s28.postimg.org/7kpdprl4d/derivation_expected_Prediction_Error.jpg)

![mean squared error](https://s3.postimg.org/lypv4bhhf/derivation_mean_Squared_Error.jpg)

# metrics

## regression metrics

**Mean squared error** / **root mean squared error** is the most common one. RMSE is used to set the scale of the metric back to the one from the outcome. If you optimize for MSE you also optimize for RMSE and vice-versa. In gradient-based methods though they will be using a different learning rate. The constant (ie having one prediction for every observation) that minimizes this metric is the mean.
$$
MSE = \frac1N\sum(y_i-\hat{y_i})\\
RMSE = \sqrt{MSE}
$$
**R-squared** compares the accuracy of our model with the constant (mean) model. If you optimize MSE you also optimize R-squared.
$$
R^2 = 1 - \frac{MSE}{\frac1N\sum(y_i-\bar{y_i})}
$$
**Mean absolute error** does not penalize large errors as much as MSE so is less impacted by outliers. It is also useful for contexts where errors have an absolute meaning. For example, if you are predicting costs of something, two errors of 5 dollars are the same as one of 10​. Best constant for MAE is the median.
$$
MAE = \frac{1}{N}\sum\mid y_i - \hat{y_i} \mid
$$
MAE vs MSE

- do you have outliers and are you sure they are outliers? Use MAE
- are extreme values just unexpected or naturally large values? Use MSE

**Mean squared percentage error** and **mean absolute percentage error** take into account the error relative to the label. For example, if we are off by 1, it matters if the label is 10 (which is a 10% error) or 100 (which would be a 1% error). They can also be thought of as weighted versions of MSE and MAE, weighted inversely to the target label. Therefore also the best constant is a weighted mean (MSPE) or weighted median (MAPE).
$$
MSPE = 100\% \frac{1}{N} \sum (\frac {y_i - \hat{y_i}}{y_i})^2 \\
MAPE = 100\% \frac{1}{N} \sum \frac {y_i - \hat{y_i}}{y_i}
$$
**Root mean square logarithmic error** is RMSE in logarithmic space. It also "cares" more about smaller values like MAPE / MSPE, but not as much. It is more often used than them (according to Kaggle course, though I've never heard of it before). It penalizes *underprediction* more than *overprediction*. 
$$
RMSLE = \sqrt {\frac{1}{N} \sum( log(y_i + 1) - log(\hat{y_i} + 1))}
$$

## classification metrics

With classification problems we have **hard predictions** (predicting a class for each row) and **soft predictions** (predicting a probability).

**Accuracy**, the % of correctly classified (hard) predictions, is usually not very useful since often times one category is much more common. Therefore, you can always predict that category and get high accuracy.

**Log loss** works with soft predictions and harshly penalizes high certainty (high probability) which leads to a wrong answer. Usually predictions are clipped so that a zero probability is replaced by a very small number, and 1 is replaced by a number very close to 1. Best constant for log loss is the proportion of categories in the dataset. Eg, if there is 90% dogs and 10% cats in the dataset, you should always predict 90% chance of a case being a dog. Below is a binary version of log loss, there is also the multiclass version.
$$
logloss = - \frac{1}{N} \sum y_i log(\hat{y_i}) + (1-y_i) log(\hat{1 - y_i})
$$
**Area under the curve** works by "trying out" all possible thresholds for classification (works only for binary problems) and calculating the % of correctly classified cases. There are as many thresholds as there are rows. (I'm not sure this is the right way to describe it, but I don't like how Kaggle course described this) It is the area of the curve when you plot the results for all those thresholds, namely their false positive (x-axis) and true positive (y-axis) rate. Both of them start at zero (no one is below the threshold) and end with one (everyone is below the threshold). Another way of explaining what AUC does is that it is the % of correctly classified pairs of objects: if you pair each object from one class with all objects from the other, in what % of cases will the model predict a higher % for the right class. The baseline score for AUC is 0.5.

**Cohen's (weighted) Kappa** compares the accuracy of the predictions with a baseline accuracy, similar to what R-squared does for MSE. It is zero if the predictions are the same as the baseline, and 1 if all predictions are correct. The baseline is the prediction of randomly permuted predictions (there is an analytical solution for this, no need to really permute).
$$
kappa = 1 - \frac{error}{baselineError}
$$
Weighted Kappa assumes some errors are more costly than others. The most common use case is when the classes are rank-ordered, for example: healthy, mild disease, severe disease. Weights can be manual or preset, eg linear or quadratic. An example of quadratic weights is below.

|         | healthy | mild | severe |
| ------- | ------- | ---- | ------ |
| healthy | 0       | 1    | 4      |
| mild    | 1       | 0    | 1      |
| severe  | 4       | 1    | 0      |

## target vs optimization metric (and other tips)

Metrics are what you judge your model performance on. Optimally, you would use the same metric in training and judging the model. But you cannot do that always. The metric usually needs to be differentiable for you to be able to train the model on it. Therefore, there is often a difference between the **target metric** (what we want to optimize) and the **optimization metric** (the metric the model will use in training).

Some metrics can be optimized directly (MSE, LogLoss) but many cannot.

- MAE: use a similar metric such as Huber loss. Looks like MSE when error is small, otherwise looks like MAE
- MPSE, MAPE, RMSLE: preprocess the training set and optimize another metric
  - these are weighted versions of MSE or MAE, so we can use sample weights and optimize for those
  - if your library doesn't support sample weights, you can manually resample using those same weights, with replacement. Though likely you will have to resample and retrain several times and average the predictions to make them more stable
  - RMSLE involves transforming the target, fitting a model using MSE, then transforming back
- accuracy: tune the threshold after optimizing another metric like Hinge loss (looks close enough to accuracy) or logloss
- AUC: sometimes you can optimize it directly (it uses pairwise losses instead of pointwise as other metrics), but otherwise optimize logloss
- Kappa:
  - (easy) optimize MSE and find the right thresholds
  - (hard) custom loss for GBM and neural nets
- any metric: write a custom loss function (depends on the algo)
- any metric: optimize another metric and use **early stopping**: use metric 2 to train, but stop the training when metric 1 (target metric) starts getting worse 

Calibrate Random Forest predictions for logloss. Calibration means correcting the predictions so that the predicted probabilities fit the true probabilities better (eg, among samples that have a predicted 20% chance of churning should be 20% churners). The probabilities might not be calibrated, even though the ranking is preserved

How to calibrate: Platt Scaling (fit a logistic regression to your predictions), Isotonic scaling (fit isotonic regression), or other stacking (fit XGBoost or neural nets to your predictions)

# validation

**Train / validation / test set goals**:

- the training set is used to train your model, ie set your model parameters
- the validation set enables you to pick your model (ie algorithm), its hyperparameters or any other parameters that are free to vary (eg, choices in data preprocessing, training set choices). Another word for this is *tuning*
- the test set "stands for" your production set and tells you how well your model will work in real-life. Another word for this is out-of-sample data, because your model has "never seen" this data, nor did you make any choices based on itt se

Common approaches for validation

- holdout set: split the training set into two parts, typically 80/20%
- leave-p-out: you divide the training set into not p (training) and p (test). You test every permutation of this. Becomes computationally intractable for large N. Leave-one-out is simpler and easier to compute, and k-folds is an approximation to leave-p-out
- k-folds means splitting the dataset randomly into k sets, and use each set as the test set with the rest as the training set. An issue for me is how to decide how many folds. Jeff Leek says smaller k = less variance, more bias, larger k = more variance, less bias. K-fold is more useful if you have a small dataset.
- leave-one-out is k-folds where $k = N$. Ie, you use only one datapoint to test the model you trained on the others. Computationally intensive. UWA machine learning course suggests that the "best approximation to the generalization error of the model" is when you use leave-one-out (I guess because the training set size here is closest to the actual size).
- random split without replacement: with infinite iterations, it approaches leave-p-out. If you do with replacement, this would be the bootstrap and it underestimates the error
- bootstrap = random split with replacement. Underestimates the error, but has a correction called bootstrap 632
- time series data requires you to divide the time into chunks. It has auto-correlation and you should not disregard this.
- use stratification when target values are imbalanced, you have a small dataset or for multiclass classification. The goal is to preserve the target distribution between train and validation sets
- tips from Kaggle course: use holdout when you get the same hyperparameters for each split, use k-fold when the differ and use leave-one-out for really small datasets

**out-of-time validation**: these approaches ignore a common issue in business: you train your data on the past and predict the future. Therefore, your validation set needs to be in the future too. A special case can also be using moving windows: train on one week, then validate on next, then move one week ahead.

If the datasets differ in some way, your validation set should ideally be as different from the training set, as the test set is from the validation set (or the combined validation + training set). You can use exploratory analysis to check this. For example, if you have a classifier, you can plot two features on the x and y axes, and color the dots with predicted class or test.

Difference between training and production data is covered under terms such as concept drift, model drift, etc. This is something I should look into more, but I understand the general idea. Production data can have 1) different population ratios (eg, more users from China) and 2) different relationships between features and the outcome. The issue under 1) can be solved by reweighting the training set (or other approaches). Issue 2) is more complicated. One approach is feature engineering. A common example is how Windows 7 might have signaled a new computer 10 years ago, but today would be old. Therefore, you transform this into "OS age".

Finally, the most important point is that validation set needs to mimic the test set, which needs to mimic the production set. And the training method used in validation needs to be the same as for test and for production.

# machine learning workflow

 1. model selection, ie selecting the tuning parameters
 2. assesing the model performance

## assessing model performance
If you choose the tuning parameter on the test data, then the parameter will be overly optimized for exactly that data. Consequently, the estimate of error on future datasets (estimate of generalization error) will be optimistic (i.e., too low). Solution is to use 2 test sets, where the one used for parameter tuning is called the validation set. The test set is used to estimate the gen. error.

How to split the initial dataset? No hard rules, typical is 80/10/10 or 50/25/25.

# gradient descent
Why use gradient descent instead of closed-form solutions?

 - there may not be a closed-form solution
 - gradient descent is usually computationally more efficient

## algorithm
```python
# hill descent algorithm for one parameter
def gradient_descent(start_value, tolerance, stepsize):
    
    derivative = derive(start_value)
    new_value = start_value
    
    while derivative > tolerance:
        # subtracting: if the derivative is positive, you want to go back
        new_value = new_value - stepsize * derivative
        derivative = derive(new_value)

    return new_value 
```

## multiple parameters
To optimize with more than one parameter you use a *gradient*, which is a vector of partial derivatives. Each parameter has a rate-of-change function in which the other parameters are treated as constants.

A gradient does not give you *one* answer on "how to change the parameters to minimize the objective function", but $p$ answers ("you need to change this parameter this much, that parameter that much...").

## a gradient at each datapoint
The objective function is defined as a sum of costs / deviation across all datapoints. The gradient question is "how much does this sum change if we change the parameters?" 

$$totalCost(params) = \sum_{i=1}^{N}(cost(datapoint_i))$$

Using the **derivatives sum rule**, a derivative of this sum is the sum of the derivatives for individual datapoints. This makes sense: the direction of change for the cost function on the whole dataset is the sum of the directions of changes on each datapoint.

$$\frac{d}{dw}(cost_1(w) + cost_2(w) ... cost_N(w))$$
$$= \frac{d}{dw}(cost_1(w)) + \frac{d}{dw}(cost_2(w)) ... \frac{d}{dw}(cost_N(w)))$$

This means that for the gradient, the data is constant, and the parameters are variable. I.e., the gradient is a sum of derivatives of functions where the datapoints are constant (so there are N such functions), and we take the derivative with respect to the parameters.

## choosing gradient parameters: stepsize and tolerance
A **fixed stepsize** can overshoot when you approach the minimum. That is why you could use a **decreasing stepsize**. The stepsize is a function of the step iteration count. But in this case you can go too slowly towards the solution. Choosing the stepsize is a finnicky thing.

 - $\large\eta_t = \frac{\alpha}{t}$
 - $\large\eta_t = \frac{\alpha}{\sqrt{t}}$

The **tolerance** is the acceptable size of the gradient: "if the gradient is this small, stop the descent". When you are optimizing across multiple parameters, you calculate the root of the squared partial derivatives to get the **gradient magnitude**. This is actually a more general idea of the [vector magnitude](http://farside.ph.utexas.edu/teaching/301/lectures/node28.html), a generalization of Pythagoras' theorem.

# coordinate descent
You minimize the objective function *feature by feature*, where you keep that one dimension variable and others are constant (ie, their last calculated value). So you turn a p-dimensional minimization problem into a series of 1-dimensional problems.

Choice of how to pick the next coordinate:

  - random (= stochastic gradient descent)
  - one-by-one

## comparison with gradient descent

 - there is **no stepsize** in coordinate descent
 - converges with strongly convex functions, whereas it doesn't for others (I don't have more info)
 - less computationally expensive

## how to asses convergence?
At each step in coordinate descent, you are minimizing against a particular feature, and the objective function gets lower and lower. You should stop when the magnitude of change, the step size, gets too small.

But you need to go through all of the features since only one of them might be significant. Therefore, you stop when `max stepsize < threshold`.

# steps in training a ML model
 - training data
 - feature engineering
 - ml model (e.g., y = a + bx)
 - ml algorithm (e.g,. gradient descent)
 - quality metric (e.g, root mean squared error)

Ranking the importance of different components (rough idea)

1. available features and dataset size
2. feature engineering
3. hyperparameter tuning
4. picking the right algorithm (most often GBM wins)

# hyperparameter tuning

## hyperparameters per model

I will write "O" for parameters that lead to overfitting if you increase them, and "U" for those that increase underfitting

GBM

- O max depth and number leaves (lightGBM): set around 7 as a starting point, take care with high values since it will increase training time
- O subsample / bagging fraction: % of samples to use for each tree
- O colsample / feature fraction (by tree or by level): % of features to use when fitting a tree
- U min child weight: very important and varies widely, I think this is related to the minimum of examples in the terminal node
- O eta / learning rate and num rounds: eta is the learning rate (ie, how much weight is given for each tree), and num rounds is how many successive trees we want to build; you can freeze eta to a small number (0.01 - 0.1) then change the number of rounds until you find the right number; roughly, if you multiply the number of rounds by 2x, you want to divide the learning rate by 2
- seed: the point here is that the seed should not affect the model too much

random forest

- n estimators: since trees are independent in random forest, there is a sweet spot of n estimators that is *sufficient* and adding more does not benefit performance
- O max depth: can be set to NA, ie infinite depth, depth 7 a good starting point (but higher than for GBM)
- O max features
- U min samples leaf: similar to min child weight
- criterion: how to evaluate the tree leaves, usually gini coefficient or entropy; just pick the one that works better
- n jobs: number of cores to use

neural nets

- O number of neurons per layer
- O number of layers: add a lot of complexity, can lead to non-convergence
- U SGD as the optimizer
- O modern optimizers (Adam / Adadelta / Adagrad...)
- O batch size: useful starting point is 32 or 64
- learning rate: does not impact overfitting, but it will impact whether the network converges at all (too high) or takes too long to converge (too low); if you increase batch size by X, you can increase your learning rate by X
- U regularization: l1/l2, dropout/dropconnect, static dropconnect

linear models: SVMs, regularized regression, SGD classifier and regressor, Vowpal Wabbit (online / out of core regression learning) has FTRL (follow the regularized leader... no clue what that is)

- U regularization parameter

## practical tips

- select which parameters to tune: find the most important ones for your model since you do not have time to tune all of them. Look at which parameters are usually tuned eg on Kaggle for your model
- know what each parameter does, or usually it is enough to know if increasing it makes the model underfit or overfit more
- there are a lot of libraries that you can use for automatic tuning: hyperopt, scikit-optimize, spearmint, gypyopt... Usually you just specify the space of parameters and it runs automatically
- manual tuning is also common
- do not spend too much time tuning parameters since feature engineering, getting more data and validation methods are most important. But I suppose having good starting points (takes experience!) is important



# tips / tidbits

## normalization

 - apply the same normalization procedure on the test data as you did on the training data. Eg, if you're standardizing the values, use the mean and the standard deviation *from training data*. An alternative is to rescale the weights.

### options

 - unit length: divide each feature by its L2-norm, ie., the sum of squares. Each feature will then have L2 = 1.
