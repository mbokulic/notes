# definitions
Time series data (ts) are *contextual data representations*, which means that attributes (variables) fall into one of two classes:
 
  - contextual attributes: time in the case of ts
  - behavioral attributes: one or more values for each contextual value

With ts, there exists a class of **online (real-time) analyses**. This means you analyze data as you get them.

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

# forecasting

## stationarity
**Strictly stationary** ts have a constant mean and variance. I.e., the distribution of values in any time window is constant.

**Weakly stationary** ts have either

 - constant covariance over time (covariance stationarity)
 - a trendline for the mean (trend stationary)

Convert to stationary using **differencing** where data at *t* is equal to difference of *t - (t -1)*.

## autoregressive models
Data at *t* is equal to a linear combination of previous time points.

## autoregressive moving average models
A moving average model predicts future behavior according to the history of deviations from forecasts (meaning this is a recursive model).

ARMA model (autoregressive moving average) combines the two. ARIMA introduces differencing into the equation.

## regression
Use predictor ts to forecast criterion ts. You need to have a forecast of the predictor in order to have a forecast of the criterion. Autocorrelation in the residuals is common here and should be addressed.

### special variables

Modelling the **trend** involves using time as the predictor. Linear terms are most common, or piecewise linear trends (linear with a break). Splines are common too. Quadratic and other poly terms can be unrealistic when extrapolated. 

Modelling **seasonality** involves using dummy variables for "seasons" (month, weekday, quartals, etc).

You can also model effects of **special periods** by including them as predictors. 

 - public holidays: dummy var
 - outliers: dummy var
 - trading days: count (e.g., nr of workdays in a month)
 - including each workday specifically (e.g., nr of Mondays in a month)

**Lagged variables** such as advertising expenditure. Effects should fall-off with time.

**Interventions** can be handled in one of three ways:

 - spikes: happen only during one period. Use dummy var.
 - permanent changes: have constant effect after time t. Use a step var
 - change of slope: piecewise linear trend

### residual diagnostic
In addition to classical issues, with ts you need to check for autocorrelation.

 - ACF plot
 - Durbin-Watson test (lag one autocorrelation)
 - Breusch-Godfrey test (higher lags)

### non-linear relationships
Same solutions as for classic regression: poly terms, splines.


# decomposition
Ts can be decomposed into these components, multiplicative (log) or additive

 - seasonal: look for seasonal patterns
 - trend-cyclic: look for long term trends (stable, upward, downward, shifts) and cycles (non-seasonal patterns)
 - remainder: all that is left

## moving averages
They estimate the trend-cyclic component. Higher order means more smoothing. You can also use MA's of MA's (e.g., 2x 4-order MA). This way you are putting more weight on the middle points more. There are also other weighting options: these lead to smoother lines.

Use MA's to **cancel out seasonality**. E.g., order 7 to cancel out weekday effects.

## classical decomposition
Steps

 - calculate trend-cycle using moving averages
 - estimate seasonal component by averaging across seasons. Transform so that they add up to zero
 - the remainder = value - trend - season

Multiplicative models involve dividing instead of subtracting.

## X-12-ARIMA
For quarterly and monthly data, based on the classical decomposition but fixes its drawbacks. Complicated stuff.

## seasonal and trend decomp. using loess (STL)
Handles any type of seasonality, allows seasonality to change over time, smoothness of trend can be controlled, robust to outliers.

## forecasting after decomposition
Forecast the seasonal and de-seasonalized components separately. For the first, the naive seasonal method is used. For the second, any method.

# exponential smoothing
Value at time t is equal to a weighted average of values at t = 1 ... (t-1). The values are weighted according to a smoothing parameter that falls off exponentially when going towards the past.

Introduces a **small lag** since value at t is equal to a combination involving (t-1).

Has **two parameters**: the smoothing parameter and the value at t = 1. The latter is usually set to the actual t=1 value, but both parameters can be estimated by minimizing SS errors.