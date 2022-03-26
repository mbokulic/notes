# calibrating probabilities

Calibrating a classifier consists in fitting a regressor (called a calibrator) that maps the output of the classifier (as given by predict or predict_proba) to a calibrated probability in [0, 1]. The samples that are used to train the calibrator should not be used to train the target classifier.

Platt Scaling is simpler and is suitable for reliability diagrams with the S-shape. Isotonic Regression is more complex, requires a lot more data (otherwise it may overfit), but can support reliability diagrams with different shapes (is nonparametric).

Platt Scaling is most effective when the distortion in the predicted probabilities is sigmoid-shaped. Isotonic Regression is a more powerful calibration method that can correct any monotonic distortion. Unfortunately, this extra power comes at a price. A learning curve analysis shows that Isotonic Regression is more prone to overfitting, and thus performs worse than Platt Scaling, when data is scarce.

Calibration should be done (I think) after picking the hyperparameters. You do it by using one set of hyperparameters, then calibrating that model by splitting the training set or using k-folds.

 - train / test involves training then calibrating on the test set. You validate on the out of sample validation set
 - k-folds would mean you split the trainset 3-5 times and train the logistic or isotonic regression... I don’t yet get how you turn this into one calibrator? ... nah. Upon more reading I see that the early text was imprecise. You calibrate always on a (single) test set. CV is just used to validate it multiple times and average the performance.

I don’t see why you couldn’t do the same procedure by calibrating each model in the grid. But maybe it wouldn’t change things much and you would lose time.

% used to calibrate is another hyperparameter.

You should always use stratified sampling in imbalanced class problems.

Does Random Forest predict calibrated probabilities?
The histograms show peaks at approximately 0.2 and 0.9 probability, while probabilities close to 0 or 1 are very rare. An explanation for this is given by Niculescu-Mizil and Caruana 1: “Methods such as bagging and random forests that average predictions from a base set of models can have difficulty making predictions near 0 and 1 because variance in the underlying base models will bias predictions that should be near zero or one away from these values. Because predictions are restricted to the interval [0,1], errors caused by variance tend to be one-sided near zero and one. For example, if a model should predict p = 0 for a case, the only way bagging can achieve this is if all bagged trees predict zero. If we add noise to the trees that bagging is averaging over, this noise will cause some trees to predict values larger than 0 for this case, thus moving the average prediction of the bagged ensemble away from 0. We observe this effect most strongly with random forests because the base-level trees trained with random forests have relatively high variance due to feature subsetting.” As a result, the calibration curve also referred to as the reliability diagram (Wilks 1995 2) shows a characteristic sigmoid shape, indicating that the classifier could trust its “intuition” more and return probabilities closer to 0 or 1 typically.

calibration

- DONE https://machinelearningmastery.com/calibrated-classification-model-in-scikit-learn/
- https://machinelearningmastery.com/probability-calibration-for-imbalanced-classification/
- https://en.wikipedia.org/wiki/Platt_scaling
- https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5074325/
- DONE https://scikit-learn.org/stable/modules/calibration.html
- https://ieeexplore.ieee.org/document/4724964
- DONE https://dataisblue.io/python/data_science/2020/02/16/calibrating-random-forest.html
- https://www.datascienceassn.org/sites/default/files/Predicting%20good%20probabilities%20with%20supervised%20learning.pdf
- https://booking.workplace.com/groups/AnalyticsCommunity/permalink/1147196538986818/
