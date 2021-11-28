# what are time series
Time series data (ts) are *contextual data representations*, which means that attributes (variables) fall into one of two classes:

  - contextual attributes: time in the case of ts
  - behavioral attributes: one or more values for each contextual value

With ts, there exists a class of **online (real-time) analyses**. This means you analyze data as you get them.

## components: trend, seasonality and cycles

**Trend** is a long-term increase or decrease, or both. A **seasonal** pattern is one that is affected by a fixed length related to the calendar, such as time of the year or weekday. A **cycle** is a repeating rise and fall that is not of a fixed frequency. Usually they are longer than the seasonal patterns and have more variable patterns.

When data have a trend, the autocorrelations for small lags tend to be large and positive because observations nearby in time are also nearby in size. So the ACF of trended time series tend to have positive values that slowly decrease as the lags increase. When data are seasonal, the autocorrelations will be larger for the seasonal lags (at multiples of the seasonal frequency) than for other lags. When data are both trended and seasonal, you see a combination of these effects.

## descriptive statistics specific to time series

By decomposing time series into trend-cycle, seasonality and remainder, you can calculate the relative impacts of each. This is useful if you have to compare different time series, eg to find the one with the strongest seasonality. See [here](https://otexts.com/fpp2/seasonal-strength.html) for more.

# data preparation

## missing values

 - linear interpolation most common
 - polynomial interpolation
 - spline interpolation

## noise removal

Binning (piecewise aggregate approximation)

Moving average smoothing: introduces lag of length w (time window). You lose short-term trends.

Exponential smoothing gives more importance to recent points. Lower lag. 

## normalization

Standardization (z-scores) and range-based (0 to 1)

## transformation

If the data show variation that increases or decreases with the level of the series, then a transformation can be useful, most commonly a **logarithm** since it is easily interpretable.

**Box-Cox transformations** is a family of transformations that includes the logarithm and power transformations (eg, square root). It's useful since you can adjust its parameter to fit your data, eg to make seasonal variations of the same size across time. When you back-transform your forecast into the original scale, use a bias adjustment if you want to add those forecasts together with other forecasts (more info [here](https://otexts.com/fpp2/transformations.html))

Forecasting is usually insensitive to transformations, but they do have an effect on prediction intervals.

[useful link](https://otexts.com/fpp2/transformations.html)

## adjustments to totals

Looking at non-adjusted totals can give a wrong impression:

- months or quarters have a different number of days
- population changes over time (eg, better to look at nr of doctors per 100k people than nr of doctors)
- money changes value because of inflation

## data reduction

Discrete wavelet transform: transforms the ts into differences between two contiguous time points. With these differences and the ts mean you can generate the ts data, but the idea is to only keep the biggest differences and reduce the size of the data. This should transform ts data into multidimensional static data, but I still do not understand how.

Discrete Fourier transform: (I do not understand completely) ts is a linear combination of sine functions with different periodicity.

Symbolic aggregate approximation: Bin the data first, then transform into discrete data, typically 3-10 breakpoints. Breakpoints are chosen so that they satisfy a particular frequency distribution.

## similarity measures
Choice of measure very important and application-specific. You need to address distortions:

 - different scales: normalize
 - temporal shifts: ts not starting at same time
 - temporal scaling: ts having different "tempos", need to stretch or compress time at different segments
 - noncontiguity in matching: some periods have very low similarity ()

Euclidean distance, for equal-sized ts and where there is a one-to-one correspondence between data points.

Dynamic time warping involves stretching and shrinking the time dimension if that leads to higher similarities between the ts.

Window based methods involve splitting the ts into segments and comparing those segments. 

Edit distance: not popular for continuous ts since allowed editing operations are hard to define.

Longest common subsequence: if you discretize ts you can get this.

# simple models
Time series are thought to be realizations of stochastic processes, which are themselves sequences of random variables. This is analogous to the relation between eg a sample (realized) and normal distribution (theoretical random variable). The difference is that time series are *sequences*. The set of all possible realizations is called an ensemble.

A stochastic process describes the properties of the random variables and how they relate to one another. Ie, it specifies the joint distribution of the full set of its random variables.

## simplest models (benchmarks)

These simple models are often used as benchmarks ([Hyndman](https://otexts.com/fpp2/simple-methods.html)).

- **average method**: take the past average
- **naive**: take the last observation
- **seasonal naive**: take the last observation of the same season
- **drift**: assume that the average change over time will continue, equivalent to drawing a line between the first and last observation

## stationarity

Think of stationarity like this: if you pick a random time, you will expect the same thing: the values anywhere will come from the same distribution. In a more formal way: the joint distribution of a group of variables with a specific spacing is the same as the group of another vars that are at a distance $lag$ from them and have the same spacing.

A weird consequence is that **cyclic time series can be stationary**, as long as the cycle is not predictable.

There is **strict and weak stationarity**, though I don't clearly get the difference. This is what I wrote before:

**Strictly stationary** time series have a constant mean and variance. I.e., the distribution of values in any time window is constant, different parts of the ts are interchangeable. This implies no seasonality and no trend. The random vars are identically distributed, but not necessarily independent. The covariance does not need to be zero, but it *only depends on the spacing between variables, not where you are in the ts*.

**Weakly stationary** ts are characterized by their properties, not as a consequence of strict stationarity (interchangeable parts).

 - constant mean
 - autocovariance depends only on the spacing btw variables => follows that variance is constant since when spacing = 0, we get the variance

Hyndman book suggests there is also **trend stationary**, characterized by a trendline for the mean (trend stationary).

### differencing
Convert non-stationary to stationary using **differencing** where data at $x_t$ is equal to difference of $x_t - (x_(t -1))$.

Differencing takes care of the level (ie, the mean) of the series, therefore it takes care of trend and (possibly) seasonality). Transformations help with the changing variance.

In case the series is still not stationary, you can do another step of differencing: **second-order differencing**. You are differencing the differenced values. Going beyond second-order is rare.

**Seasonal differencing** means subtracting the last value from the same season. Eg, subtracting May 2018 from May 2019.

The outcome of differencing might be noise:

- you get white noise if you difference a random walk model
- you get noise + constant trend if your model corresponds to the simple drift method
- if you get noise after seasonal differencing, then your data corresponds to the naive seasonal method

**How much differencing should you do?** There isn't a clear formula for this, though you could use tests that check for stationarity (see below). When both seasonal and first differences are applied, it makes no difference which is done first—the result will be the same. However, if the data have a strong seasonal pattern, it's better to do the seasonal difference first since it could make you stop right there. If you do differencing first, the seasonal pattern will still remain.

- KPSS test is significant if there is a unit-root present, ie the time series is not stationary. Use `ndiffs` to calculate the required number of differencing (or `nsdiffs` for seasonal differencing)

## autocovariance
Covariance between $x$ at time $t$ and $x$ with a lag $k$. Same for autocorrelation.

$$
autocov_k = cov(x_t, x_(t - k))
$$

The pure random time series is an example of a stationary ts and below is its autocorrelation graph.

```r
pure_random = ts(rnorm(100))
acf(pure_random)
```

## random walk
The random walk model posits that each following value is equal to the previous value plus noise. It is an accumulation of noises from previous steps.

It is not a stationary process as both the mean (if it is not zero) and the variance change.

The random walk is not a time series but a stochastic process. A realization of the random walk *is* a ts.

$$
x_t = x_(t - 1) + e_t
e ~ Normal(mean, variance)

x_1 = 0
x_t = \sum_{i=1}^{t} z_i 
$$

```r
random_walk = 0
for (i in 2:1000) {
    random_walk[i] = random_walk[i - 1] + rnorm(1)
}
random_walk = ts(random_walk)
plot(random_walk)
acf(random_walk)
```

### detrending the random walk
If you remove the trend from a random walk you will get a stationary time series.

```r
random_walk = 0
for (i in 2:1000) {
    random_walk[i] = random_walk[i - 1] + rnorm(1)
}
random_walk = ts(random_walk)

# detrending
plot(diff(random_walk))
acf(diff(random_walk))
```

## moving average models (MA)
A moving average model of order q supposes that $x_t$ is a linear combination of its noise and the noise of lag $q$. Autocorrelation of MAQ processes has correlations up to lag Q.

MA is weakly stationary. It will have a non-null correlation between two variables where their lag is equal to or lower than $Q$. This correlation only depends on the lag, not on the position in the time series.

$$
x_t = z_t + w_1 z_(t-1) + w_2 z_(t-2) ...
$$

When lag is equal to zero, then autocovariance is equal to the sum of squared weights multiplied by the variance.

$$
/sigma^2 /sum{i=0, q}/beta_i^2
$$

More generally, with lag $k$, the autocovariance is equal to a sum of the products of weights. The larger the $k$ (ie, the larger the lag), the less there will be of these weights and the lower the covariance. With the covariance for $k = 0$ (ie the variance) we can also estimate the correlation.

$$
\sigma^{2}\sum ^{q-k}_{i=0}\beta_i \beta_{i+k}
$$


```r
noise=rnorm(10000)
ma_2=NULL

# generate an MA(2) process
for(i in 3:10000){
    ma_2[i]=noise[i]+0.7*noise[i-1]+0.2*noise[i-2]
}
# Shift data to left by 2 units
moving_average_process=ma_2[3:10000]

moving_average_process=ts(moving_average_process)
par(mfrow=c(2,1))
plot(moving_average_process, main='A moving average process of order 2', ylab=' ', col='blue')
acf(moving_average_process, main='Correlogram of a moving average process of order 2')
```

# forecasting models

## exponential smoothing

**Simple exponential smoothing**: value at time t is equal to a weighted average of values at t = 1 ... (t-1). The values are weighted according to a smoothing parameter that falls off exponentially when going towards the past.

$y_{t+1|t} = \alpha(\alpha -1)y_t + alpha(\alpha -1)^2y_{t-1} + ... $

(my interpretation) The parameter $\alpha$ should be selected by an optimization method to reduce the sum of squared errors. Then you are treating exp. smoothing as a model where each value is estimated, including the first one. Therefore besides $\alpha$ you need to estimate the forecasted first value, which is called level-0 $l_0$ (but can be set to the actual first value). Then each value can be described like below, where each value is expressed as a weighted average of the previous value, and its forecast. Or, effectively, it's a weighted sum of the previous value and all others before it.

$y_{t+1|t} = \alpha y_t + (1 - \alpha) y_{t|t-1}$

And the $t = 2$ value includes the $l_0$ parameter

$y_{2|1} = \alpha y_1 + (1 - \alpha) l_0$

The forecasted values after the last observed value are flat, ie they are all the same and are equal to the forecast of the last value. The only difference is that the further you go, confidence intervals are bigger. The forecasts for observed values have a lag since value at t is equal to a weighted average that puts the biggest weight on $t-1$.

All exponential smoothing models have a forecast and level equation. Double and triple add trend and seasonality respectively.

- forecast: $\hat{y} = l_{t-1}$
- level: $l_t = \alpha l_t + (1 - \alpha) l_{t-1}$

### double exponential smoothing

Single exponential smoothing only has a level parameter. Double smoothing adds a trend.

- forecast: $\hat{y}_{t+h|t} = l_t + hb_t$
- level: $l_t = \alpha l_t + (1 - \alpha) (l_{t-1} + b_{t-1})$
- trend:$b_t = \beta (l_t - l_{t-1})+ (1 - \beta) b_{t-1}$ 

And the forecasted value is simply the level plus trend, where the trend is multiplied by the number of look ahead steps. This means that the last observed trend is continued indefinitely.

This constant trend is usually an overprediction. You can introduce a **dampening factor** and reduce the trend the more far ahead you look. Check the Hyndman book for details. It seems like you should be using this most of the time, but then again, you probably shouldn't be predicting too far ahead in the first place.

### triple exponential smoothing

This adds **seasonality**. The formulas are lengthy, so I will avoid retyping them here. The forecast is now comprised of the level, trend and the last observed seasonal estimate. And there are three corresponding smoothing parameters.

Seasonality can be **additive** (expressed as the part you add to each forecast) or **multiplicative** (expressed in relative terms). Multiplicative is more appropriate if the seasonal part changes with the level of the series. What this means (I think) is that it makes sense if you expect a bigger (absolute) seasonal effect if the value is higher. You can decide between them by comparing error estimates (eg RMSE).

This method can also be damped.

### state space models (ETS: error, trend, seasonality)

These are a generalization of exponential smoothing models. You get to them by re-arranging the smoothing formulas and get to the fact that every prediction is the last prediction (last level) and alpha times the residual (the error) if you use that last level as the prediction. So alpha controls how much you correct for the error, how much you adjust to the new value. You can add trend and seasonality to these equations.

- measurement equation: $y_t = l_{t-1} + \epsilon_t$
- state equation: $l_t = l_{t - 1} + \alpha\epsilon_t$

If you specify the error distribution, this is a complete model. In addition, the **error can be multiplicative**.

Another way of thinking about this is that there is a hidden layer of the model (the state: level. trend and seasonality) and the visible layer (the measurements themselves).

### model selection

Using AIC or BIC. Multiplicative models are not useful when the data can be zero or negative.

### predicting using ETS

Can generate prediction intervals unlike exponential smoothing: prediction intervals take into account the variability of the values, not just variability of the sampling. Therefore, they are wider. They tell you where the forecasted values can be found, whereas confidence intervals tell you where their *expected* values (the means) may be lying.

## ARIMA models

ARIMA means autoregressive integrated moving average model. Since this model is very opaque, I will describe it in simple language and you can check the formulas in Hyndman ([link](https://otexts.com/fpp2/non-seasonal-arima.html)) or the Practical Time Series course on Coursera ([link](https://www.coursera.org/learn/practical-time-series-analysis)).

An autoregressive model means that a datapoint is a linear combination (plus intercept) of previous time points. The order $p$ determines how many lags you go back. Moving average models model values as a linear combination of *errors*.  The also have an order denoted by $q$. These two models are in some cases interchangeable. An $AR(p)$ process can be rewritten as $MA(\infty)$ and vice-versa. This can only be done if they satisfy conditions, most importantly that their coefficients are $< 1$. If this was not true, then lags far into the past would be more influential than recent ones (check the literature if you want details). If an $MA(q)$ process can be rewritten as $AR(\infty)$ we say it is **invertible** (but not the other way around).

The stationarity and invertibility of the ARIMA model are related to the unit roots of the equation. This is some mathematical complexity which is solved by the R code. Check the literature if you want to know more.

The full specification of an ARIMA model involves 3 parameters and a constant: the two orders mentioned previously and the order of differencing. Thus, we write ARIMA models as $ARIMA(p, d, q)$.

Some regularities:

- higher $d$ leads to prediction intervals that increase in size when you want to forecast further into the future. With $d = 0$ your prediction intervals will be the same as the standard deviation of historical data
- If $c=0$ and $d=0$, the long-term forecasts will go to zero
- If $c=0$ and $d=1$, the long-term forecasts will go to a non-zero constant
- if $c=0$ and $d=2$, the long-term forecasts will follow a straight line
- If $c≠0$ and $d=0$, the long-term forecasts will go to the mean of the data
- If $c≠0$ and $d=1$, the long-term forecasts will follow a straight line
- If $c≠0$ and $d=2$, the long-term forecasts will follow a quadratic trend
- to obtain cycles, parameter $p$ must be $p >= 2$

### seasonal ARIMA

Seasonal ARIMA includes 3 more terms: $p, d, q$ , but for seasonal (lagged) values. Otherwise the process is the same.

### picking ARIMA models

You would use an automatic search procedure to find the best model, like the `auto.arima` function in the `forecast` package in R. You could also do it manually by looking at ACF (auto-correlation) and PACF (partial auto-correlation) plots. Check [here](https://otexts.com/fpp2/non-seasonal-arima.html) for more info. In most cases the results should be the same. Sometimes they might be different because `auto.arima` uses shortcuts. Use the parameters `stepwise=FALSE` and `approximation=FALSE` to make it search through all possible combinations.

A rough description of how this works is:

1. Plot the data and identify any unusual observations.
2. If necessary, transform the data (using a Box-Cox transformation) to stabilise the variance.
3. If the data are non-stationary, take first differences of the data until the data are stationary.
4. Examine the ACF/PACF: Is an ARIMA(p, d, 0) or ARIMA (0, d, q) model appropriate?
5. Try your chosen model(s), and use the AICc to search for a better model. AIC should only be used on models of the same differencing order.
6. Check the residuals from your chosen model by plotting the ACF of the residuals, and doing a portmanteau test of the residuals. If they do not look like white noise, try a modified model.
7. Once the residuals look like white noise, calculate forecasts.

`auto.arima` takes care of the steps 3-5.

And you should still test the model using an out-of-sample test set. [Check here](https://otexts.com/fpp2/seasonal-arima.html) for an example. Normally, this is the most important test: in practice you would use the best model even if it the residuals weren't stationary and normally distributed.

### prediction intervals

The prediction intervals for ARIMA models are based on assumptions that the residuals are uncorrelated and normally distributed. You should check the ACF and histogram of the residuals to be sure.

As with most prediction interval calculations, **ARIMA-based intervals tend to be too narrow**. This is because only the variation in the errors has been accounted for. There is also variation in the parameter estimates, and in the model order, that has not been included in the calculation. In addition, the calculation assumes that the historical patterns that have been modeled will continue into the forecast period.

### picking between ETS and ARIMA

They are often interchangeable. You can't use AIC as they are different model, but you can use cross-validation or train/test splits.

## regression

Use a predictor time series (one or more) to predict another time series. If you want to forecast in the future you need to have a forecast of the predictor in order to have a forecast of the criterion. Autocorrelation in the residuals is common here and should be addressed (how?)

### special variables
Modelling the **trend** involves using time as the predictor. Linear terms are most common, or piecewise linear trends (linear with a break). Splines are common too. Quadratic and other poly terms can be unrealistic when extrapolated.

Modelling **seasonality** involves using dummy variables for "seasons" (month, weekday, quartals, etc). Another option involves using **Fourier series** (also known as harmonic regression). These are pairs of sine and cosine terms that are proportional to the season length. You can use them instead of dummy variables, but they are much more common with very long seasons like weeks. In those cases you usually need less of them than dummy variables. Also, they are useful if you have multiple seasonality patterns.

You can also model effects of **special periods** by including them as predictors.

 - public holidays: dummy var; Easter is special since it's on different days each year and lasts more than one day
 - outliers: dummy var
 - trading days (e.g., nr of workdays in a month): count them and use as a predictor
 - including each workday specifically (e.g., nr of Mondays in a month)

**Lagged variables** such as advertising expenditure, you would enter as variables advertising for last month, last two months, etc. Effects should fall-off with time (a complex topic).

**Interventions** can be handled in one of three ways:

 - spikes: happen only during one period. Use dummy var.
 - permanent changes: have constant effect after time t. Use a step variable
 - change of slope: piecewise linear trend, a trend that bends at the time of intervention and hence is nonlinear

### similarities to non-time series regression

- non linearities resolved using poly terms, splines or transformations (you can transform the outcome too), trend can be nonlinear
- variable selection done using cross validation or the scores like AIC (although I guess there are options that go beyond those that @forecastingHyndman mentions)
- multicollinearity is an issue, though less so if you want to forecast
- if the future values of the predictors are outside of your current range, extrapolation is difficult (a general issue with prediction)

### regression and non-stationarity

Non-stationary time series are more likely to lead to spurious correlations: most obviously with time series trending upwards. The residuals of regression models using spurious correlation are likely to exhibit autocorrelation.

It is not clear to me why this is so...  @forecastingHyndman doesn't say how to address this problem. It seems to me then that it's better to address non-stationarity by introducing trend as a predictor. Simple "it's going up" predictors would then lose their predictive power.

### forecasting with regression

**Ex ante** forecasts mean actual forecasting, past to future, ie using only the data you have. Here you have to first make a forecast of the predictors or get them from somewhere else (think weather forecasts). This is often harder than just making the forecast of the univariate outcome time series.

**Ex-post** are forecasts that are made using later information on the predictors useful for studying the behaviour of forecasting models. They are useful if you want to see if the uncertainty in the model or in forecasts of the predictors is leading to bad performance.

**Scenario based forecasting** means that you pre-select the values of your predictors. It answers "What would happen if..." Regression based models are very useful for this kind of forecasting.

Using **lagged predictors** means using a predictor from the past to predict the future value. You then don't have to forecast the predictors as in ex-ante forecasts. Makes sense if the predictors have *lagged effects*.

But generally, regression is not as good for forecasting as univariate models (ARIMA, exponential smoothing), although adding some regressors in those helps (eg, holidays). Univariate models seem to work well because *the information about unobserved predictors is encoded in past observations*.

## dynamic regression

These models combine regression and ARIMA. They do so by fitting a regression, but allowing the residuals to be correlated. Those residuals are modeled as coming from an ARIMA process and only the ARIMA residuals need to be white noise.

Usually all of the predictors and outcome are differenced together.

### forecasting

As described in the regression section, you need to forecast the predictors first.

The prediction intervals will be narrower than those of ARIMA, but that's because the model will not take into account the uncertainty around the predictor estimates. Hyndman says they "should be interpreted as being conditional on the assumed (or estimated) future values of the predictor variables". But doesn't that make them useless as forecasts, except when doing eg scenario forecasts?

### special cases

#### modeling stochastic and deterministic trends

A trend can be modeled by adding $t​$ as the variable. This is called a deterministic trend: you don't expect it to change.

A stochastic trend is modeled by using differencing. The point estimates might be the same, but the prediction intervals are much wider with this one. Therefore, for longer time horizons, a stochastic trend is advised.

#### harmonic regression

If the seasonality is long (think above 24), then it is advised to model it using Fourier series as regressors and ARIMA for the residuals. The more complex the seasonality, the simpler will the ARIMA model on the remaining variation be. Check [this link](https://otexts.com/fpp2/dhr.html).

### lagged predictors

Commonly used with advertising data where it is assumed that marketing spending has an effect on the current period but also $k$ periods ahead. You can pick $k$ using AIC or cross-validation. Make sure you use the same training dataset: if your maximum lag is 4, then you can only use the data from the 5-th month, otherwise you cannot calculate data for $x_{t - 1}$.

I guess that more complicated models also use weights that drop off with time.

## neural networks

This model (at least as described by Hyndman) is equivalent to an auto-regressive model, but non-linear. The input layer consists of lagged values of order $p$, and seasonally lagged values of order $P$. These are usually picked by fitting an AR model and picking its parameters. If you use one hidden layer, the neural network AR model has 3 hyperparameters, with $k$ standing for the number of hidden nodes: $NNAR(p, P, k)$.

Given that it is non-linear, it can asymetric cycles. The way to calculate prediction intervals is by introducing errors (normal or bootstrapped) and re-calculating the model.

## bootstrapping and bagging

From your existing time series you simulate similar ones, reshuffling them to some extent using existing error (see below). Then you forecast them (eg using ETS) and average those point forecasts to obtain a bagged forecast which is usually more precise than the original one. Requires more processing power as you are fitting more than one model, plus for simulating the time series. You can also use this procedure to create more accurate (and wider) prediction intervals, but this will take even more time as you need more simulations.

The way the time series are simulated is unique, since you cannot just draw errors from the existing residuals (as in classic bootstrapping). First, the time series is Box-Cox-transformed, and then decomposed into trend, seasonal and remainder components using STL. Then we obtain shuffled versions of the remainder component to get bootstrapped remainder series. Because there may be autocorrelation present in an STL remainder series, we cannot simply use the re-draw procedure that was described in Section [3.5](https://otexts.com/fpp2/prediction-intervals.html#prediction-intervals). Instead, we use a “blocked bootstrap”, where contiguous sections of the time series are selected at random and joined together. These bootstrapped remainder series are added to the trend and seasonal components, and the Box-Cox transformation is reversed to give variations on the original time series.

## ensemble methods

Involves using multiple models then averaging them. Usually works better than any of the individual models.

## residual diagnostic

Each observation in a time series can be forecast using all previous observations. We call these **fitted values. Fitted values always involve one-step forecasts although you may want to use multi-step forecasts yf your forecast horizon is long (I'm not 100% sure about when and why you would do this). Fitted values are often not true forecasts because any parameters involved in the forecasting method are estimated using all available observations in the time series, including future observations. 

You calculate **residuals** by subtracting the true value from the fitted one. You want the residuals from forecasting to satisfy these points:

1. uncorrelated
2. unbiased, mean equal to zero
3. normally distributed
4. constant variance

If 1 and 2 are not satisfied, you can always improve your forecast. But if they *are* satisfied it doesn't mean you can't.

You may not be able to improve on 3 and 4. This will hurt your prediction intervals.

Tools for detecting autocorrelation

 - ACF plot 
 - Durbin-Watson test (lag one autocorrelation)
 - Breusch-Godfrey test (higher lags)
 - Ljung-Box test (most commonly used)

Prediction intervals give you the likely range in which new values will appear. They fan out as you forecast longer horizons. You can get them by assuming normally distributed errors or using bootstrapping: adding errors drawn from existing residuals.

Prediction intervals tend to be too narrow. This is because they only take random error into account, whereas there are 3 more sources of uncertainty (2nd and 3rd related to the process underlying the data). You can 

1. The parameter estimates
2. The choice of model for the historical data (does this mean eg ARIMA vs ETS, or setting their hyperparameters?)
3. The continuation of the historical data generating process into the future

# evaluating forecast accuracy

The accuracy of a forecast is different from its diagnostics. Diagnostics are made on training data (ie, data used to build the model), whereas accuracy is done on test data (ie, out-of-sample data).

How to test the accuracy of your model depends on your **forecasting context** (ie, what you want to forecast). 

## train-test split and cross-validation

A good way to choose the best forecasting model is to find the model with the smallest RMSE computed using time series cross-validation. Cross-validation involves splitting the time series into pre- and post- periods multiple times and testing on the post-periods. If you want your forecasts to be accurate for a long term forecast, use t + step lags ([more info](https://otexts.com/fpp2/accuracy.html)).

Usually the test is at minimum of the length of the forecast horizon.

Train-test split means you only split the data once, whereas cross-validation means splitting as many times as possible with the given lag.

## accuracy (error) measures

Accuracy measures always aggregate the prediction errors in some way.

**Scale dependent** measures differ based on the scale of the units and cannot be used to compare the prediction of models of different units

- mean absolute error (MAE) leads to forecasts of the median
- root mean squared error (RMSE) leads to forecasts of the mean, which is why it is more commonly used

**Unit-free measures** can be used to compare forecasts on different datasets

- mean absolute percentage error (MAPE)
  - impossible to calculate when the outcome is zero
  - problematic with near zero values
- symmetric MAPE (sMAPE) that corrects for over-penalizing negative errors in MAPE
  - Hyndman argues against it
- mean absolute scaled error (MASE) which compares the forecast to a simple forecast: naive or seasonal naive. It divides the mean absolute error of a forecast with the simple one, therefore 1 means you're doing no better than it is doing


# decomposition
Time series can be decomposed into components:

 - seasonal: look for seasonal patterns
 - trend-cyclic: look for long term trends (stable, upward, downward, shifts) and cycles (non-seasonal patterns)
 - remainder: all that is left

These can be additive or (more commonly, esp with economic stuff that I look at) multiplicative. An alternative is to first transform the data until the variation appears to be stable over time, then use an additive decomposition.

A simpler decomposition involves just removing the seasonal component, thus producing seasonally-adjusted data.

## moving averages
They estimate the trend-cyclic component. Higher order means more smoothing. You can also use MA's of MA's (e.g., 2x 4-order MA). This way you are putting more weight on the middle points, but there are other weighting options. Generally, weighing lead to smoother lines.

Use MA's to **cancel out seasonality**. E.g., order 7 to cancel out weekday effects.

## classical decomposition
Steps

 - calculate trend-cycle using moving averages
 - remove trend-cycle from original by substraction or division
 - using the remaining values, estimate seasonal component by averaging across seasons. Transform so that they add up to zero
 - the remainder = value - trend - season (or divided by trend x season)

Objections to classical decomposition

- does not work well with rapid changes
- assumes that the seasonal component stays the same through time
- trend-cycle or remainder is not available for the first and last few observations because of moving average calculation

## other decomposition methods

- X-12-ARIMA: For quarterly and monthly data, based on the classical decomposition but fixes its drawbacks.
- SEATS: uses ARIMA, similar results to X-12
- STL (seasonal and trend decomposition using loess): handles any type of seasonality, allows seasonality to change over time, smoothness of trend can be controlled, robust to outliers. But can only deal with additive decomposition

## forecasting after decomposition
This is done by forecasting the seasonal and de-seasonalized components separately. 

- for seasonality something simple is used, like the naive seasonal method
- for trend and remainder, you can use any method. Very commonly ETS is used

# specific topics

## forecasting hierarchical time series

Many time series are aggregations of others and in many cases you want to have forecasts for both the lower and higher levels. Example: sales of a company may be divided into sales per country, and those even further into sales per region. Another case is **grouped time series**, where multiple hierarchies exist. To use the same sales example, we might divide  sales further into product types, a categorization that is *crossed* with the existing regional one. When forecasting these multi-level time series, you want them to be **coherent**. The lower levels should sum up to the higher levels.

There are three simple ways in which you can do this:

- **bottom-up**: you forecast the lowest level, then sum them up to the higher ones
- **top-down**: you forecast the top level, then proportionally divide it among lower levels. There are several ways in which to calculate these proportions. The most involved one described in the Hyndman book involves first forecasting the lowest level, using those forecast to calculate the forecast proportions, then applying them to the top level
- **middle-out**: you start at the middle, then use the top-down approach for lower levels and bottom up for higher

The most state-of the-art approach seems to be **optimal reconiliation**. (following text is from [here](https://www.firstanalytics.com/single-post/2017/07/17/How-to-Slice-It-Using-Optimal-Reconciliation-for-Hierarchical-and-Grouped-Forecasts)) First, independent forecasts are generated for all nodes at every level of the hierarchy, and then an optimal reconciliation step is used to adjust the forecasts. The reconciled forecasts are a weighted sum of the forecasts from all nodes, where the weights are found by solving a system of equations with an aggregation matrix that indicates the node relationships throughout the hierarchy. I will skip the math on this one as it is involved, but I wonder if the resulting matrix with the weights could be used to tell where among the levels is the "interesting" stuff happening. Could it be a method for discovery?

## multiple seasonalities and other related

In some cases you will observe more than one seasonality, and most typical approaches (ARIMA, ETS, ...) can deal with only one. Example: you have hourly data and want to include hour-level seasonality and weekday-level.

Options

- use ETS on multiple seasonally adjusted data decomposed by STL (decomposition by loess)
- dynamic harmonic regression using Fourier terms for multiple seasonalities, these can even include covariates
- TBATS automatic models that use dynamic harmonic regression but allow seasonality to change over time

**Weekly seasonality** is an issue on its own since it is not an integer: a year has 52.18 weeks. It is also long, meaning a lot of parameters would be needed. Ways of approaching it include the ones described above.

**Moving events** like Easter or Chinese New Year should be modeled as dummy variables, but not a lot of models support that. The only choice (so far, for me) is dynamic regression.

## vector autoregression: bi-directional forecasts

Use this if you want to model variables that are thought to be influencing one another. Each variable gets modeled as an AR process involving itself and lags of other variables.

It is also used in testing whether one variable is useful for forecasting another in Granger causality tests.

## forecasts of counts

If your data can only take on integer values (eg, number of items sold), then strictly speaking you shouldn't be using a model that outputs continuous variables. But if the counts are large enough (>100) it doesn't matter. When it matters is when the counts are small and often zero. This kind of data is often called **intermittent demand**, ie demand that occurs only sometimes.

A simple method for doing this is **Croston's method.** It involves dividing the time series into two parts:

- demand: all non-zero values
- inter-arrival time: the time between non-zero values

Then you forecast these two independently. The total forecast is then demand / inter-arrival time (ie, average demand through time).

This is something that I could've used in Monolith for estimating the number of visitors in a shop.

## **constraining forecasts**

Sometimes you would want to limit the range of forecasts to $[0, \infty)$ or some other range. You can do this using transformations (see [Hyndman](https://otexts.com/fpp2/limits.html)).

## forecasting aggregates

This would involve forecasting the year from monthly forecasts. Point estimates are easy to get by summing, but the prediction intervals less so due to the correlations between forecast errors. The way to go is to simulate sample paths from the forecasts and get prediction intervals from these.

## very short time series: how many datapoints do you need?

There is no general answer to this question as it depends on the number of parameters in the model (you need at least that many datapoints, in practice more) and the noisiness of the data (the more noise, the more datapoints).

The problem with short series is that you do not have enough data for out-of-sample validation and even cross-validation. You need to resort to AIC which tends to prefer models with a small number of parameters.

## very long time series

Real data do not come from time series models. The issue with very long time series is that the difference between the model and the true process becomes more apparent. You could use a model that can change over time (ETS or ARIMA). Or you could throw away old observations if your forecasting horizon is not big.

But one problem is not addressed in Hyndman: what if there is a break in the process? Eg, a change in the demand for a product because of a competitor.

## test set using one-step forecasts

If your forecast horizon is one step, you may want to calculate the error on your test set using only one step forecasts. The first step uses only training data, but the next steps would use more and more of the test set while still keeping the parameters trained on the training data.

## outliers and missing values

Both of these produce few errors if they are random and do not reflect something true in the underlying model. You can impute missing values using interpolation and replace outliers with interpolated values. Outliers make problems for many of the models and this is a good way of approaching them.

Otherwise, you need to model them, eg with dummy variables in dynamic regression.





















