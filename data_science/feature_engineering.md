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
 - ordinal encoding by frequency of categories (not sure, is it the same as label encoding)

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

Random ideas

- fractional part of price (eg, 2.99$ into 0.99) or another value. Eg, this can be useful to distinguish bots from humans sometimes (in a bidding humans will use round numbers, etc).

## categorical feature interactions

Useful for non tree-based models



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

## extracting features from text

**Text preprocessing**

- lowercase
- lemmatization: take a single meaning of the word (democratic -> democracy)
- stemming: take the word stem (democratic -> democr)
- removing stopwords, common words that do not have an important meaning (words like “the” or “an”) or other very common words (there you can have a threshold that says if a word is more common than X, then kick it out)

**Bag of words** is a simple approach where you have one column per each unique word in the dataset (after preprocessing like stemming, etc). Scaling options for bag of words:

- term frequency: divide the number of occurrences with the total number of words per text (per row)
- inverse document frequency: divide sample size with how many texts the word appears in, then take the log of it. Words that appear less often will have a higher score
  $log(\frac{N}{frequency})$
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























