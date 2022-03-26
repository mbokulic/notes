# general

Adapt feature engineering to the model you will use.

# feature preprocessing

## numeric features

**Normalization (variable scale)**

- no normalization needed: tree-based models
- have to use normalization: kNN (because of similarity calculation), linear models that use regularization (Lasso, Ridge regression), neural networks (because of gradient descent)
  - you can consider scaling as one of the hyperparameters (eg if you do z-values or [0, 1] min-max scaler)
- normalization not necessary, but it will affect interpretation of coefficients: linear regression, logistic regression

**Outliers**: solving these is important for linear models

- Use winsorisation (maybe treat it as a hyperparameter?). Use the same bounds that you used in the train data to test data?
- Rank transformation. Use the rank mapping from train also on test, or concatenate train and test when calculating the ranks.
- treat them as missing values (eg, if a person says their age is 550)

**Transformations using log or sqrt.** Useful for linear models and neural nets

**Combining models using different preprocessing** can be a useful way to increase the variation of the models in stacking. Also, you can concatenate data using differently preprocessed data, though I don't understand this fully: concatenate columns (makes sense) or rows (doesn't make sense to me).

## categorical features

Categories need to be turned into numbers for most algorithms to be able to ingest them.

**One hot encoding** involves replacing each category with a column that has 1 if the row has the target category, ie you have to add as many categories as there are unique values. Similar to dummy variables, though they have N-1 columns (relevant for linear models where you would have perfect collinearity otherwise).

 * High cardinality features are bad for trees. They will generate a lot of columns, with most of them being irrelevant and full of zeros
    * they are unlikely to be picked for a split (not sure where I got this)
    * these features will overshadow numerical columns in number and thus reduce the "efficiency with which they are used". I guess it means it will slow down the training by a lot
    * they will make the column randomization hyperparameter (*colsample_bytree*) work weirdly. The categorical column will always be present (because it is split into so many individual columns), while the non-categorical columns will almost never appear together.
 * It may lead to out of memory errors in case it creates too many columns. The solution is sparse matrices, where available. They store only non-zero values.
 * not an issue for H2o since it doesn’t one hot encode
 * (but there is no solution) I guess the solution is to pick only the top N categories

**Label encoding.** Turn category values into numbers of a single column. In most cases it doesn't work, since numbers imply an ordering. But it can work with ordinal values (eg, economy / business class in a plane, max education level, etc) or other variables where being close in number implies being close on the target. With linear and NNs it probably won't work.

**Frequency encoding.** Encode values with their relative frequencies. It can work with linear and tree models, only if the frequency is correlated to the target in some way. In case there are ties in the frequencies the feature will not distinguish between different values. You can apply a rank transformation that removes ties in that case.

**Target encoding**: replace category labels with mean target value per category. Benefit is it doesn’t add dimensions to the dataset like one-hot encoding. Drawback is it can overfit since you could be basing the mean on small sample size. A popular way is to use cross-validation and compute the means in each out-of-fold dataset, h2o does it (I dont understand how that works). Another is **additive smoothing**: you use the overall mean along the category mean and you smooth it out with a single parameter. See below or in the sources
$$
\begin{equation} \mu = \frac{n \times \bar{x} + m \times w}{n + m} \end{equation}
$$
other options

 * Label encoding where you choose an arbitrary number for each category. Usually useless.
 * Vector representation a.k.a. word2vec where you find a low dimensional subspace that fits your data. Require some fine-tuning and don’t always work out of the box
 * Optimal binning where you rely on tree-learners such as LightGBM or CatBoost. “Relying on LightGBM/CatBoost is the best out-of-the-box method”
 - ordinal encoding by frequency of categories

sources:
 * https://roamanalytics.com/2016/10/28/are-categorical-variables-getting-lost-in-your-random-forests/
 * https://maxhalford.github.io/blog/target-encoding/

## missing values

- Replace with a constant like -1, 999, etc. Works with trees, but not linear models. With categorical features it can work with both.

- Replace with mean or median can be used with linear models or NNs. For trees it's better to use another option
- Add is_null column (also in combination with replacing with mean/median), useful for trees and NNs

- Feature value reconstruction eg if there is a linear relationship between time and the metric, or something in that vein. Not very common.
- Remove rows with missing values (I guess if there are only a few of them, or if those rows have a lot of missing values)

Features with missing values are a big issue when you use them to generate new features. The newly generated features can be misleading. If you use target encoding (or encoding categorical features with anther numeric feature), best to ignore missing values (or start replacing them *after* feature generation).

Category values missing from training data: turn them into missing values or use eg frequency encoding

Some tree-based models can handle missing data automatically (XGBoost, H2o random forest and GBM, CatBoost)

# feature generation

Use prior knowledge / critical thinking and exploratory analysis to come up with these.

**Ratios.** Price per square meter for apartments.

**Multiplications / formulae.** Air distance (straight line distance) calculation.

**Categorical feature interactions.** Eg, economy class and gender. Useful for linear models which would otherwise ignore this.

Other

- fractional part of price (eg, 2.99$ into 0.99) or another value. Eg, this can be useful to distinguish bots from humans sometimes (in a bidding humans will use round numbers, etc).

## mean encoding

The basic version of mean encoding involves creating a new feature based on a categorical variable where the values are the mean target values. Eg, say that you have a binary classification task where you predict whether a user converted. Users from US convert in 5% of cases. Therefore, the mean encoding feature for US is equal to 0.05.

Other options include:

- weight of evidence $ln(\frac{good}{bad}) * 100\%$
- count of positive cases
- difference: count positive - count negative

**Motivation**: Mean encoding splits the cases by target value really well. Works especially well with tree-based models which have trouble with high cardinality variables. Diagnostics: if it takes you a long time to start overfitting when you increase tree depth, this means that you need a lot of splits to extract information from your data. Try mean encoding in that case.

Main issue is target leakage which leads to overfitting. It is easiest to see in the extreme case: if there is only one example of eg users from Suriname, then the mean encoding for this category will be equal to the target. This happens for categories bigger than N = 1, but less obviously.

Fixing target leakage:

- CV loop regularization: estimate means on the samples outside the fold. In k = 5, for the 1st fold, you use samples from folds 2-5 to estimate the mean encoded categories. Fill NAs with the global mean (for categories that appear only in one fold). There is still target leakage though, especially with leave-one-out

- smoothing: combine category and global mean, inversely proportionate to the size of the category. Parameter $alpha$ controls the amount of regularization, with $alpha = 0$ meaning no regularization. In a way, $alpha$ is the "size of the category that we can trust". You have to use another method alongside this one, for example the CV loop method.
  $$
  meanEncoding = \frac{mean(target) * nrows + globalMean * alpha}{nrows + alpha}
  $$

- adding noise: you degrade the quality of mean encodings. An unstable method, not clear how much noise to add (another hyperparameter you need to tune). Usually used with leave-one-out

- expanding mean: sort the data and when you calculate mean encoding for row i, use only rows until i. This makes the feature of unequal quality (lower rows have worse quality). The way to fix this is to reshuffle, retrain and average the predictions. This introduces the least amount of leakage and does not have hyperparameters to tune. Built in in Catboost. 

- recommended by Kaggle people: CV loop and expanding mean

summary of correct validation

- for tuning stage:
  - estimate encodings on train
  - map to train and validation
  - regularize on train
  - validate model
- for testing stage: same, but now train and validation are one set (train set) and test takes the role of the validation from before

Beyond means:

- for regression tasks you can use percentiles (like the median), SD, distribution bins ("how many between 1-10, 11-20...")
- for multiclass tasks: if you have k classes you will have k mean encoded features
- many-to-many relations: eg we have data on which apps users use; first we calculate mean-encodings per app, then each user gets these encodings as a vector (eg if you have app 1 and 2 you get those two values), then we take statistics from these vectors (mean, min, max...)
- time series: can do complex things like take the result from the previous day, or mean spent per category so far per day (check the video for more ideas)
- binning numeric features: you bin the feature then treat as categorical -> therefore you can mean-encode it. The features that would benefit from this are those that are used in many splits of the tree model (you can also use those split points to pick the bins)
- interactions: you pick two categorical features and combine them (can be done for more than two features as well). How you select them: if they often appear in two neighboring nodes in a tree (you can pick the most frequent ones). Catboost does some version of this automatically

In general, if there are a lot of categorical variables it is useful to try mean encodings and interactions between those variables.

## feature interactions

For categorical features, create new categories by concatenating old ones. Or alternatively, one-hot encode old and multiply those encodings. You can also bin numeric features and treat them like this.

Numeric features can be multiplied, but also (less commonly) summed, subtracted, divided.

Feature interactions explode the feature space and can lead to overfitting. You can minimize this with feature selection or dimensionality reduction.

- feature selection example: fit a random forest on all interactions, then keep only the most important ones by feature importance

Interactions are very effective for tree-based methods.

Higher order interactions (those between 3 features or more) immensely explode the feature space and are usually done manually 
("art more than science"). An automatic way of doing something close to this is to use decision trees. You take the index of tree leaf as a category ("this row is in the 25th leaf"). Kaggle people say that with Random Forest you can use *all* of the trees, but that feels like too much. Maybe better to train a simple decision tree and use those indices?

## time-based features

**Seasonality.** Month, season, day of week, hour...

**Time since**

- row independent: eg time since 1970
- row dependent: time since last holiday, last weekend, last sales campaign...
- relating two dates: time between last purchase date and subsequent visit to website (or whatever, that's not a good example)

**Linear trend.** Say you want to predict sales in various shops. And there is a linear trend through time -> sales go up. A linear model will pick that up easily, you just add a feature that encodes time (eg month or week from t = 0). This will not work with tree-based models as they will make the split based on existing time periods and ignore the new ones in the test set. Example: say you have 20 weeks of data in the train set. When predicting for eg week 25, the decision tree will use the average for week 20. Tree-based models are bad at extrapolating.

## location-based features

**Distances** Between the object and nearest interesting elements

- if you have locations of important points: between an apartment and a hospital, center of the city, landmarks...
- if you do not have locations: distance between *this* apartment and the best apartment in the district, or if you divide the map into squares within the square this apartment belongs to. Or cluster the data and use cluster centers as important points.

**Aggregation by location**

- average apartment price within eg 500m
- number of apartments within 500m

**Use Lat and Lon directly.** A decision tree can find splits within those, but you might have to rotate the coordinate system. Not sure how you pick the angle... treat it as a hyperparameter?

## features based on statistics and distance

This I can only explain on examples.

Statistics examples:

- aggregations for the city a user comes from: number of users visiting from this city, median basket size for the user from that city, etc
- page visits and ads: max and min price of an ad for the page visited by users, SD of prices, most visited page

Distance / neighbors: use when we do not have explicit grouping. Most natural example is physical location (described elsewhere), but you can create your own distance space. For example, users similar to their purchase history. Examples:

- mean target of 10 / 20 / N closest neighbors
- mean distance to 10 closest neighbors, or neighbors with target 1 or 0

## matrix factorization techniques

Techniques such as: SVD, PCA, truncated SVD (for sparse matrices), non-negative matrix factorization (NMF) good for count data and for tree-based models

Be careful to use the same transformation on the test as you used on train

I do not have a good source for this, will need to find it.

t-SNE is a special such technique that does this in a non-linear fashion. Has a hyperparameter called perplexity which influences the results by a lot. Requires a lot of time to train, therefore dimensionality reduction is often done before.

## extracting features from text

**Text preprocessing**

- lowercase
- lemmatization: take a single meaning of the word (democratic -> democracy)
- stemming: take the word stem (democratic -> democr)
- removing stopwords, common words that do not have an important meaning (words like “the” or “an”) or other very common words (there you can have a threshold that says if a word is more common than X, then kick it out)

**Bag of words** is a simple approach where you have one column per each unique word in the dataset (after preprocessing like stemming, etc). Scaling options for bag of words:

- term frequency: divide the number of occurrences with the total number of words per text (per row)
- inverse document frequency: divide sample size with how many texts the word appears in, then take the log of it. Words that appear less often will have a higher score
  $log(\frac{N}{appearances})$
- These two approaches are usually combined into one. Then it's called TFiDF
- There are other approaches similar to TFiDF

**N-grams** are combinations of N words. N-grams can then be used in the same way words are used

**word2vec** involves finding vector representations for words so that their meaning is preserved. It involves (afaik, from the NLP course) counting how many times words appear to each other within a range of X words (typically 2 or 3). Look into the NLP notes for more details.

## using errors in the data

you may find unusual data combinations in the dataset, for example a case which has more clicks than impressions on a website. In this case you can create a dummy variable that codes for whether there was an error or not.

## feature grouping

Explore correlations between features or their means (after normalization). Can you spot groups of features? Can you aggregate them in some way?

## extracting feature from images

I guess the state of the art is to use a pre-trained convolutional network (trained on similar images) and fine-tune it to your problem. These nets turn images into a series of  more and more abstract vectors, culminating in a classification. I won’t write more than this since the Kaggle course did not go into more details.

# issues connected to validation

## unseen values in the test set

You may have values in the test set (and the production set) which you did not see in the training set. To estimate the impact of this, you can report performance separately on rows with unseen values. If they differ a lot, you may want to kick out the feature, or create a separate model for these rows.

Another issue (that I've seen no one talk about) is when the outcome range differs from train and test set. Tree-based models (which are commonly used) in particular can suffer from this since they do not extrapolate.

## feature drift

https://towardsdatascience.com/drift-in-machine-learning-e49df46803a























