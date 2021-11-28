**leverage measure**: how strong is a value's potential to influence the model. Whereas outliers have strange y values, high leverage points are the ones with strange x values, or strange combinations of predictors (more info here)

**High leverage points:**
Center City is said to be a "high leverage" point because it is at an extreme x value where there are not other observations. As a result, recalling the closed-form solution for simple regression, this point has the potential to dramatically change the least squares line since the center of x mass is heavily influenced by this one point and the least squares line will try to fit close to that outlying (in x) point. If a high leverage point follows the trend of the other data, this might not have much effect. On the other hand, if this point somehow differs, it can be strongly influential in the resulting fit.

**Influential observations:**
An influential observation is one where the removal of the point significantly changes the fit. As discussed above, high leverage points are good candidates for being influential observations, but need not be. Other observations that are not leverage points can also be influential observations (e.g., strongly outlying in y even if x is a typical value).

- If a data point has high leverage and doesn't fit the model, it might be an error in data entry
- ?influence.measures gives you various measures of influence that will (generally) tell you how strongly would the model change if you delete a particular element. Check the references there if you want to investigate further. To give a taste:
    - cooks.distance gives back the sum of squared differences between the predicted values with the full model vs the model without one observation
        - high values indicate you need to check those observations. How high? D > 4/n can be used, but better to plot them
    - dfbetas is the difference in coefficients if you leave the point in the analysis vs remove it
    - hatvalues is the potential for influence. You can express each fitted value as a weighted sum of all response values, where the weights are the hat values. Sum of squared hat values gives you the overall hat value for a single observation.
- plots hat values vs studentized residuals vs Cook's distance: influencePlot(fit, id.method="identify", main="Influence Plot", sub="Circle size is proportial to Cook's Distance" )




plot (lmObject) to get a bunch of useful graphs.

- Useful link that explains some of these
- explains heteroskedasticity and some plots
- another good resource, goes through different violations and shows plots

fitted vs residuals plot (plot(lmObj, which = 1))

- http://stats.stackexchange.com/questions/82682/how-do-i-interpret-this-fitted-vs-residuals-plot
"... you can use Tukey's nonadditivity test to see if a transformation might help"
"...it's not unusual to see a downward-sloping edge in a residuals-vs-predicted plot; it happens when there is a frequently-attained lower bound (e.g., zero) on the y values"

- http://stats.stackexchange.com/questions/116370/what-kind-of-residual-plot-does-this-variable-have
"... residuals all lie above a line with negative slope. The key question to answer is whether the response is positive by definition, or at least non-negative. If so, negative predicted values don't make much sense, and a logarithmic transformation or using a logarithmic link seems strongly indicated."

- http://stats.stackexchange.com/questions/76226/interpreting-the-residuals-vs-fitted-values-plot-for-verifying-the-assumptions

qq-plot (which = 2)

- http://stats.stackexchange.com/questions/101274/how-to-interpret-a-qq-plot
- QQ plot: see if the error term is normally distributed. How to interpret? One approach is to simulate random normal variables then compare your QQ plot to theirs (link)
- another great explanation, with good heuristics

scale-location plot (which = 3)

- fitted values vs squared residuals. I do not see what this adds to the fitted vs residuals plot

residuals vs leverage (which = 5)
plot the standardized residuals (i.e., a measure for outliers in y) vs leverage (measure for outliers in X). The observations that are unusual in X have high leverage, and if they are also outliers they will influence your fit

- good explanation with plots
"... The model is badly distorted primarily in the fourth case where there is a point with high leverage and a large (negative) standardized residual."

posible causes of plots that are not "pretty"

- heteroskedasticity
- missing model terms

remedies

- heteroskedasticity can be remedied by using a concave transformation (e.g., log or sqrt)
- sometimes heteroskedasticity can point to violations of the linear model, and can be remedied by including interaction terms or higher order polynomials
- if each response is an average of n raw observations and n is not constant across responses, you can use weighted least squares where you weight by n (ISL, p 96)
- color the residuals by other variables to see if there is a relationship
- plot the residuals ordered by another variable to see the same, or even order them by the row number to see if something is missing
- use the sandwich package that contains the "agnostic models"; the function vcovHC takes the lmObject as the input

check whether there is a linear trend - plot a scatterplot of x-y, or better plot the residuals since they show violations more clearly and can be plotted for multiple predictors

- use Poisson regression
- use data transformation
- fit a nonlinear trend (e.g., polynomial regression)
- interpret just the linear trend (and vcovHC to estimate the significance values and confidence intervals)

with outliers, identify them on normal plots, or run a for loop (i in 1:N) that will conduct a linear regression without each data point, then compare the b-value without that point to the original b-value. ISL calls outliers in y outliers, and outliers on the predictors high leverage point

- several types of outliers:
    - outliers in y typically have little effect on the regression equation (ISL, p96), but if they are also high discrepancy points (meaning that they are outliers in y conditional on their x values) have a strong effect on the r-squared and the standard errors of the coefficients
    - high leverage points have strange values in x. They reduce the standard error of the coefficients
- delete outliers if they are mistakes
- calculate student residuals (residual / standard error) with rstudent(), then plot them and classify values where |studentized residual| > 3 as outliers
- run sensitivity analyses if you think the outliers are real values
- use a robust linear model (e.g., rlm from the package MASS); they downweight outliers

collinearity makes the regression coefficients unstable

- variance inflation factors (VIFs): adding predictors to a model increases the standard errors of the remaining predictors. It increases their variance, and VIF is a measure of how much, compared to when the new predictors are orthogonal. More specifically, VIF = var(beta) with the full model / var(beta) with only the one predictor. Range is 1 - Inf, and 5 - 10 is considered a high VIF
    - function vif, package HH (there's also one in package car, I didn't check if they are the same)
    - the more correlated the new predictor is with the old, the higher the inflation
    - if r = 0, there is no inflation
    - collinearity can be solved either by excluding one of the variables with high VIF, or by aggregating them into one variable
    - useful article on what influences VIFs