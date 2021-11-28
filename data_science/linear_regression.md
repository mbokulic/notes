# linear regression model
The outcome is a weighted sum of predictors, i.e., a line equation.

$$
\large y_i = w_0 + w_1x_{1i} + w_2x_{2i} ... + e_i \\\
\large \Sigma(e_i) = 0
$$

 - $w_0$ is the intercept, the value of $y$ when $x = 0$
 - $w_k$ is the slope, the change in $y$ for one unit of change in $x$ (but see below)

## interpretation of coefficients
Interpretation of features is always considered *while keeping other features constant*. For one unit of change in the feature, you get *weight* changes in the output. This also means that the interpretation of weights (and their magnitude) depends, of course, on the rest of the features in the model: because they are the ones kept constant in this interpretation. Interpretation is always *in the context of the model*.

With polynomials, you cannot hold others fixed, so you don't interpret individual coefficients. The same with interactions.

## the generic feature extraction equation
More generically, linear regression is a weighted sum of functions of our predictors. This equation accomodates all sort of transformations of the predictors: polynomials, interactions, trigonometric functions... The $X$ below is the matrix of predictors, whereas $h_j$ is a function of them.

$$\large y_i = \sum_{j=0}^{p}(w_jh_j(X_i)) + \epsilon_i$$

The $h_j(X_i)$ above refers to feature extraction, ie transforming the input features in some way. Features are a function across any number (or all) of our inputs variables. You can use powers, as in **polynomial regression**. Or trigonometric functions, when for example modelling a trend in time.

## quality metric: residual sum of squares (RSS)
The residual is the deviation of the predicted outcome from the actual outcome. Squared residuals means that larger deviations are penalized disproportionately more.

The goal of the machine learning algorithm (e.g., gradient descent) is to minimize this deviation.

```python
def predict(features, weights):
    # prediction is a dot-product of the N x p feature matrix
    # and the weights p x 1 vector
    return numpy.dot(X, w)

def objective(predicted_outcomes, true_outcomes):
    residuals = true_outcomes - predicted_outcomes
    # same as np.asscalar(np.dot(np.transpose(residuals), residuals))
    # ie, you multiply the residual vector with its transpose
    return np.sum(residuals ** 2)
```

## gradient of RSS
Gradient of RSS is the change of RSS with respect to changes in its parameters. It tells us how much we have to change the parameters to minimize the RSS.

You calculate the gradient by taking the partial derivative of the quality metric with respect to each parameter. *Partial* means that a change in each parameter is observed with others held constant. A real world analogy would be standing on a hill and answering how much should you move on the axis North-South and on the axis East-West to go down. You look at each axis *separately and at the same time*, not one by one.

The gradient is therefore a vector of equations with the current state of the parameters as values. With a dataset with only one feature plus the intercept, the gradient amounts to:

$$
\begin{pmatrix}
-2\sum_{i=1}^{N}(y_i - (w_0 + w_{1}x_i))\\\
-2\sum_{i=1}^{N}(y_i - (w_0 + w_{1}x_i))x_i
\end{pmatrix}
$$

The $w_1$ parameter is weighted by $x_i$, meaning that the gradient will be more under the influence of datapoints that have high $x_i$. This makes sense, as changing the weight will have the most "repercussions" (i.e., the effect of the highest magnitude) on such datapoints. When $x_i = 0$, changing the weight has no effect.

Actually, the gradient to the first parameter, the intercept, is the same as for the slope if you take into account that $x_i$ is always 1.

The general form with is presented below. $H$ represents the matrix of functions of features. RSS is defined as the dot product between residual vectors, which amounts to the sum of squares. The result is a vector of partial derivatives.

$$
\bigtriangledown RSS = \bigtriangledown [(y - Hw)^T(y - Hw)]\\\
= -2H^T(y - Hw)
$$

## closed form
By setting the partial derivatives in the gradient to zero, you can derive a closed-form solution. Ie, you calculate the weights directly instead of having to rely on an iterative algorithm like gradient descent.

$$
w = (H^TH)^{-1}H^Ty
$$

You first get the dot product of the predictor matrix and its transpose. The dimensions are `p x N` (transpose) `N x p`. The resulting matrix is `p x p` and it is invertible in most cases if $N > p$. More accurately, N should be the nr of linearly independent observations.

Complexity of the closed form is $O(Np^2 + p^3)$. $Np^2$ is for calculating the matrix x matrix-transpose. "One pass" in that equation means one multiplication, as in $x_ix_j$. $p^3$ is for calculating the inverse.

# regularized models: ridge and lasso
Their goal is to balance model fit (you want low bias) and model complexity (you want low variance)

If you want to avoid overfitting, you should cover each possible combination of the predictor and outcome values.

 - this is harder if you have a small dataset
 - and harder still if you have a large number of predictors

Overfitted models generally have very large weights. These two models penalize larger weights. Generally, both have this form `objective = measure of fit + measure of the magnitude of coefficients`

 - [link](http://stats.stackexchange.com/questions/64208/why-do-overfitted-models-tend-to-have-large-coefficients) to why

## ridge regression

### objective function
RSS plus lambda times L2 norm squared (sum of the squared coefficients). L2 norm is the square root of the sum of the squared coefficients (eg, Euclidean distance).

$$
objective = RSS + \lambda \lVert {w} \rVert_2^2 \\\
= (y - Hw)^T(y - Hw) + \lambda w^Tw
$$

If lambda = 0, then when minimizing this function the resulting parameters will be equal to the least squares parameters (ie, ordinary linear regression).

If lambda = infinity, then if you want the cost to be lower than infinity the parameters need to be zero. `y = 0 + epsilon`.

This means that by setting the lambda parameter, you are making the weights be somewhere between the least squares weights (high variance, low bias) and zero (lowest variance possible since model is always the same no matter the predictors, high bias).

### gradient
Since the derivative of a square is $\frac{dy}{dx}x^2 = 2x$, the derivative of the l2 norm is as shown below.

$$
\bigtriangledown objective = -2H^T(y - Hw) + 2\lambda w
$$

Weight-by-weight, it looks like below. Remember that $h_j(x_i)$ is actually the predictor $x_j$ for the i-th datapoint, only expressed as a function of the complete input vector $x_i$.

$$
w_j^{t+1} = w_j^t - \eta(2\lambda w_j^t - 2\sum_{i=0}^{N}h_j(x_i)(y_i - \hat{y_i})) \\\
= (1 - 2\lambda\eta)w_j^t + 2\eta\sum_{i=0}^{N}h_j(x_i)(y_i - \hat{y_i})) \\\
= (1 - 2\lambda\eta)w_j^t - \eta partial[j]
$$

The 2nd expression and the 3rd expression show that at each step $t + 1$ we are first shrinking the old weight, since $2\lambda\eta$ is a positive number, and 1 minus that expression will be lower than 1. After the shrinking we fit the weights to the data.

### closed-form
Very similar to the closed form of ordinary linear regression: if you set $\lambda = 0$, you get its closed form. The computational complexity is the same, but the resulting matrix is invertible *always* when $\lambda > 0$. It is said that the lambda term is making the resulting matrix "more regular". Therefore, ridge regression is said to be a *regularization technique*.

$$
w = (H^TH + \lambda I)^{-1}(H^Ty)
$$

### do we penalize the intercept?
It does not make sense to make the intercept smaller, as a higher intercept does not indicate overfitting. The way we implement this non-penalization mathematically is in one of two ways:

 - the identity matrix is modified so it has a zero at the $[1, 1]$ position. This means that nothing will be added on the account of $w_0$.
 - center the data so that the outcome has a 0 mean

## lasso regression
Lasso enables you to discard some features from the model by setting their weights to zero. This has computational and interpretational benefits.

### objective function
Lasso uses the L1 penalty, which is a sum of absolute values. As with ridge regression, you do not include the intercept in the penalty term.

$$
RSS + \lambda \left|\left|w\right|\right|_{1}
$$

#### comparison with ridge regression
The lasso objective function will lead to **sparse models**, which means that features will be knocked out of the model, ie, their weight will be set to zero.

Assuming a high penalty, with ridge regression you will keep all of the features but their weights will be very small. With lasso you will keep only a few of them and their weights will be a lot larger compared to the ridge regression coefficients.

The L2 penalty contours are ellipsoids, whereas L1 penalties are rhomboids (in 2D that would be a diamond). Rhomboids are "pointy" and the points are where some weights are set to zero. The UWA Coursera course have a great illustration on this which I will not reproduce here.

Why not just use ridge regression and set a threshold for the coefficient magnitude? When features are correlated, ridge regression will lower their weights to avoid the penalty. This could lead us to discard important features with the thresholding scheme. I think the reason is more general: you cannot use thresholds since weights change when features are knocked out of the model.

### gradient
Lasso regression diverges at this point: the absolute value function does not have a defined derivative when `x = 0`. This is why you use **subgradients**.

#### coordinate descent calculation
In coordinate descent, you optimize the objective function feature-by feature. Before applying it to lasso, here is how coordinate descent works for least squares regression.

##### least squares coordinate descent
Assume that you normalized each feature to unit length, ie divided each feature by its L2-norm. The coordinate descent means that we are searching for the minimum of the function where $w_j$ is variable and other weights are constants. We start by splitting the derivative into the variable and invariable part.

![IMG_0192.jpg](https://s2.postimg.org/8s70alxhl/IMG_0192.jpg)

We then set derivative to be zero.

$$
-2 \rho_j + w_j = 0 \\\
w_j = \rho_j
$$

$y_i - \hat {y} (\hat {w}_{-j})$, the second part of $\rho\_j$ is actually the residual without feature j. So $\rho\_j$ on the whole is the residual *without the feature* times the feature, ie the correlation (I don't think that's correlation, but ok). If this correlation is high, this means that there is some variance left in the residuals that the feature predicts, meaning that it has importance (ie, weight) for the model. If its low, or the residuals are low themselves, then the weight will not be high.

##### lasso coordinate descent
Coordinate descent in the lasso case uses an if clause. If $\rho_j$ is between $-\dfrac {\lambda } {2}$ and $+\dfrac {\lambda } {2}$, then the new weight is set to zero.

Keep in mind, this assumes normalized features. If unnormalized, you would divide the result with the L2 norm of the feature vector.

$$
w_j = \begin{cases} \rho_{j}+\dfrac {\lambda } {2},\rho_{j} < -\dfrac {\lambda } {2}\\\ 0\\\ \rho_{j}-\dfrac {\lambda } {2},\rho_j>\dfrac {\lambda } {2}\end{cases}
$$

This leads to the **soft thresholding** effect of lasso.

![lasso vs least squares vs ridge coefficients in relation to rho](https://s13.postimg.org/oya3d76if/Selection_026.jpg)

This formula depends on the notion of **subgradients**. When you cannot calculate the gradient, you can still find a set of planes that are a lower bound for a function. These planes, defined by their slope, are the subgradient. Looking at the image below, you know the gradient when $X$ is not equal to zero (it is -1 or +1). But when $X=0$, you get a set of answers $[-1, 1]$ which are the subgradients.

I will not go into the derivation here but just write the gist of it. As before, you set the gradient equal to zero to calculate the weight. The difference here is that you get a different answer (a different solution to the equation) depending on whether the resulting weight would be below zero, above zero or equal to zero. For it to be equal to zero, the gradient needs to be in the set of subgradients. And the fact that this is a set (instead of a single solution) is what leads to soft thresholding.

![subgradients](https://s30.postimg.org/49bu8p76p/subgradients.png)

#### other lasso solvers

 - least angle regression and shrinkage (LARS), older algorithm
 - parallel and distributed models
     + parallel coordinate descent
     + parallel stochastic gradient descent
     + parallel independent sou
 - alternating direction method of mulitpliers

### closed-form
There is no closed-form solution for lasso.

### practical issues

#### interpreting selected features

 - what is selected depends on what you put in
 - sensitive to correlated features. Small changes in the data can lead to one feature being selected vs the other not
 - results depend on the algorithm used (lasso vs subset selection, etc)
 - when you raise the penalty, a feature might pop back in contention because another feature was pushed out (I am not 100% sure of this since they did not mention this explicitly)

#### cross-validation leads to low lambda
Cross-validation will not give you an optimal lambda size for doing model selection. I am not yet sure what this means as CV does lead you the most predictive model. Why would lambda need to be higher for model selection, I am not sure. They say "read Kevin Murphy's book for more details".

#### debiasing the lasso: lasso as step 1
A common procedure is to use lasso as the first step, removing the features whose weights lasso set to 0, then doing an unregularized least squares regression on the ones left. The resulting solution will have lower bias than the lasso solution.

#### elastic net: blending ridge and lasso
Two issues with lasso:

 - when features are highly correlated, lasso tends to select arbitrarily amongst them (high variance btw results). And you often want to include all of them together.
 - ridge regression has been shown to have better predictive performance in most cases

**Elastic net** includes both L1 and L2 penalties together. Zou and Hastie(2005) suggest that elastic net has better predictive performance than lasso, but still provides sparse models. It has a grouping effect in that correlated features are in or out of the model together. And, it performs better when the nr of predictors is much larger than the nr of observations.

# subset selection models

## all subsets
Try out all subsets of features: no features, all models with just one feature, two features... all up to a model including all of the features. For each model size you pick the best subset. These best subset are not necessarily nested. Eg, the best 2-feature model does not need to contain the feature from the best 1-feature model.

There are typically too many such models. If $p$ is the nr of predictors, there are $2^{p+1}$ of model permutations (`+1` is for the intercept). The way you get to this number is by noticing that a particular subset is defined as a vector with 0s and 1s of length `p` where a 0 means that a particular predictor is not in the model and vice-versa (see below for an illustration). For only 30 predictors that is already over a billion permutations.

$$
\left[0000\right] \\\
\left[1000\right]
$$

## greedy algorithms: stepwise selection
These algorithms are more computationally efficient than trying out every subset of predictors. The forward selection goes like this:

 - start with an empty set of features, or just the intercept
 - fit the model
 - select next best feature
 - keep selecting the next feature until done

Since you are forced to keep the last best feature, the k-sized model selected with stepwise selection need not be the same as the one selected when looking at all feature subsets.

Use cross-validation to select the parameter $k$, the size of the model.

Complexity is $O(p^2)$ since you make $p$ steps and at each step you consider $p - step order$ models. So actually somewhat less than $O(p^2)$. This is if you go through all of them, without stopping, eg, when the cross-validation error starts rising.

# nearest neighbor regression
This is a non-parametric approach. These approaches can fit very complex functions if they work with a lot of data. The motivation is that this approach offers very flexible models, models that are very localized to each subset of the data.

Non-parametric means that there is no model. There are no parameters that you estimate. There are very few assumptions made about the underlying true function.

Other examples of non-parametric models: splines, trees, locally weighted structured regression models.

## k-nearest neighbors algorithm
For each observation `i` you find k most similar observations defined by some similarity metric on the features. `k` is the tuning parameter for kNN regression. The lower the `k`, the more flexible the algorithm: lower bias but higher variance.

Your prediction for observation `i` is the average across those observations.

### Voronoi tesselation / diagram
This is a 2D representation of the similarity borders for when `k = 1`. Each region contains one observation and the border denotes the x-y coordinates where that observation is the closest / most similar observation.

![Voronoi tesselation](http://www.cs.wustl.edu/~pless/546/lectures/f16_voronoi.jpg)

### issues with kNN

 - boundary issues: the resulting prediction is constant at the boundaries since there aren't enough neighbouring observations so all boundary points get the same nearest-neighbor set. This is worse for large `k`
 - sparse region issues: ragged, discontinued predictions where there are few observations
 - the predictions are not smooth, there are discontinuations / jumps when neighbors get in or out of the set. This might be fine for predictive accuracy, but the resulting predictions might be nonsensical (eg., you get a big jump by slightly changing one feature)

These issues are solved with weighted kNN and kernel regression.

## weighted kNN
More similar neighbors get more weight when calculating the prediction. The predicted value is the weighted sum where $c_ij$ is the weight for observation $i$ for its j-th neighbor.

$$
\hat {y}\_{i}=\dfrac {c_{ij}y_{j}+\ldots c_{ik}y_{i}} {\sum \_{j=1}^{k}c_{ij}}
$$

Weights for more similar neighbors should be higher: the function `f(distance)` decays with distance magnitude. A simple method is taking the $1/distance$. 

More generally, we use **kernels**. A kernel is a similarity function, meaning that its inputs are two observations and its output is a single value ([SO link](http://stats.stackexchange.com/questions/152897/how-to-intuitively-explain-what-a-kernel-is)). The most common case, **isotropic kernels** are a function of just the distance between two observations. They typically have a parameter that controls the rate of decay. A common one is Gaussian kernel.

## kernel regression
This is a variant of kNN where k = N, ie you use the whole dataset when calculating the predicted value. Other name for this is Nadaraya-Watson kernel weighted averages.

Often times not very different from weighted kNN since some kernels have a bandwidth. They only take observations within this bandwidth into account. The higher the bandwidth, the lower the variance but higher bias. The choice of this bandwidth has a much greater influence than the choice of kernel in most cases.

## local constant fit vs local regression
kNN and kernel regression are examples of locally constant fits. Ie, they  fit a constant function (a very inflexible model), but locally. Kernel regression is a combination of constant fits, *one for each point*. Another name for this is *locally weighted averages*.

But you can also do a locally weighted linear regression, ie fit a line or a polynomial at each point. This helps smooth out the function even more, but has a higher variance. In most cases local linear regression is the best.

 - local linear fit reduces bias at the boundaries with a minimum increase in variance
 - local quadratic fit doesn't help with boundaries and increases variance, but helps capture the curvature of the interior
 - with enough data, local polynomials of odd degree dominate those of even degree

The image below represents the bias that a locally weighted average type of algorithm (an Epanechnikov kernel regression) has at the boundaries and at the curvature. The blue line is the true function and the green is the fitted one.

![kernel regression](https://s22.postimg.org/a5ai8u34h/kernel_regression.png)

## predictive accuracy
Non-parametric models in general function **very well with a lot of data**.

If you have *noiseless data* (no error term), with infinite data the 1-NN nearest neighbor algorithm has zero error. Ie, the model has no bias, it is infinitely flexible.

With noisy data, the MSE (the reducible part of total error) will also limit to zero with infinite data, but only if you allow k to grow. When you have noise, including more observations in the calculation allows you to smooth the error.

**Performance with little data or a lot of dimensions is poor.** Your data should cover the whole input space, and this is hard with low N and/or high p. You need an order of $N = O(exp(p))$ data depending on the dimensionality of your data. In these cases, parametric models are more useful.

## computational complexity
These models are computationally intensive. For each k-NN query (searching for neighbors for a single observation), you need to perform calculations (comparisons) on the order of $(O(Nlogk))$, or $(O(N))$ in the case of 1-NN. This is linear complexity, but can still be very intensive when N is large.