# key concepts

Survival analysis analyzes the **time to event**. Some examples: how long does it take for a couple to divorce from when they got married? Time to churn for a subscriber of a newspaper. Etc.

You visualize survival rates with a **Kaplan-Meier curve** that has time on the x-axis and survival % on the y-axis. This is just a descriptive visualization, not a (parametric) model.

Some datapoints are typically **censored** which means you do not know their exact time to event. Most common is right censored, meaning you know the event occurred *after* some time (eg, you didn't get divorced yet). But left censored (event occurred before a time) and interval censored (between some time) can also happen. Most models assume **censoring is non-informative**, ie not related to the probability of event occuring.

**Truncation** happens when the dataset is biased. Right truncated: subjects where t > x are missing. Left truncated: missing where t < x.

## symbolic represenation

**Outcome** has two parts: time and a binary value yes/no
$$
y = Surv(t, e)
$$
**Survival function**: p to survive beyond time $t$
$$
S(t) = p(T > t)
$$
**Hazard function**: probability for the event to occur in the "next few seconds" (ie, the $\delta$ value) given that you survived until time $t$ (I am not sure that the $t$ goes there, but I guess it should). Makes sense when comparing hazards, not much otherwise. This is the rate of decrease of the survival curve (similar to the derivative).
$$
Haz(t) = p(T < t + \delta | T > t)
$$
**Hazard ratio**: ratio between two hazard values (eg males vs females)
$$
HR=\frac{Haz(t, x=1)}{Haz(t, x=2)}
$$

# modeling survival

| model                   | type                                            | can estimate survival? | can estimate hazard ratio? | continuous covariates? | biggest drawback                                             |
| ----------------------- | ----------------------------------------------- | ---------------------- | -------------------------- | ---------------------- | ------------------------------------------------------------ |
| Kaplan-Meier            | non-parametric                                  | yes                    | no                         | no                     | limited covariates, no hazard ratio                          |
| exponential             | parametric                                      | yes                    | yes                        | yes                    | often unrealistic, assumes constant hazard                   |
| Weibull                 | parametric                                      | yes                    | yes                        | yes                    | still unrealistic, assumes hazard increases/decreases proportionally with $t$ |
| Cox proportional hazard | semi-parametric (intercept free to vary at $t$) | no                     | yes                        | yes                    | cannot estimate the survival function                        |

## Kaplan-Meier

Not really a model, but rather a descriptive statistic. It is just the probability to survive until time $t$, calculated as the proportion. There is no formula.

Cannot estimate the hazard ratio because it is a step function. In most places there is no change in hazard between $t$ and $t + \delta$, for a small $\delta$.

Can only include categorical covariates, and only a few (because of the combinatorial explosion).

## exponential hazard

We have events occurring independently over time with a constant rate $\lambda = y / t$ (= number of events across a period of time). Then the Poisson distribution models the number of events happening within time. 

The Exponential distribution models the time to event.
$$
f(t) = p(T = t) = \lambda e^{-\lambda t} \\
F(t) = p(T <= t) = 1 - e^{-\lambda t}
$$
The second function above is connected to the survival function, but you have to subtract it from 1. This models survival, and the lambda is the **constant hazard**.


$$
S(t) = e^{-\lambda * t} = e^{-Haz * t}
$$

The lambda/hazard is estimated using regression. It is linear in the logarithm. The term $b_0$ is similar to the intercept in linear regression. It stands for the log-hazard of the reference group at $t=0$ and it is constant with time.
$$
Haz = e^{b_0 + b_1x_1 ... + b_nx_n} \\
ln(Haz) = b_0 + b_1x_1 ... + b_nx_n
$$


## Weibull

The difference from the exponential model is that the hazard increases or decreases proportionally in time. If $\alpha$ is one, then the Weibull is the same as the exponential. If it is below one, the hazard decreases, and vice versa.
$$
\hat{b_0} = ln(\alpha)ln(t) + b_0
$$


## Cox proportional hazard

Hazard can fluctuate with time, does not have to be constant or constantly increasing/decreasing. The way it is implemented is that the reference hazard $b_0$ changes. The way it changes is non-parametric so the $h_0$ below is not a specified function, it just stands for this imagined function. The innovation is that you can estimate the coefficients (the parametric part) without specifying the baseline (non-parametric part).
$$
\hat{b_0} = ln(h_0(t))
$$
So the whole hazard function looks like, and you can see that the baseline hazard changes through time.
$$
ln(Haz) = ln(h_0(t)) + b_0 + b_1x_1 ... + b_nx_n \\
Haz = h_0(t) + e^{b_0 + b_1x_1 ... + b_nx_n}
$$


You can estimate the covariate coefficients without having to specify the $h_0$ function. But since you do not estimate $\hat{b_0}$, you cannot estimate the hazard. Ie you cannot answer the question of "how likely is it for this person to survive", you cannot do prediction.

An important limitation is that the hazard ratio is constant. Eg, if males are 2x more likely to die, this remains the same through time. This may not work depending on the problem.

## machine learning approach to Cox proportional hazard

Cox proportional model is when using the below approach and logistic regression. But you can use other models (random forest, lasso, gbm) and then it's a more general apprach.

Bin the time and explode each observation. An example: say you are modeling time to buy and you bin time into months. Then an observation that a user bought something after 5 months means you get 5 rows (t = 0, t = 1...) where the last row has a positive outcome (bought) and the other rows have a negative outcome. Time is a dummy variable.

The model will learn the hazard function. To understand why, here's an example. If you look at only rows where $t=3$, you will find users that have not bought until that time (ie, they "survived"). The probability that they buy at $t = 3$ is the conditional probability $p(y|T >= t, X)$ (X stands for the covariates).

Binning time allows us to use hazard rates to express the full survival. You read this as "chance to survive until time $t$ is equal to the chance of surviving in time 1, time 2... up to $t - 1$"
$$
S(t) = (1 - Haz_1)(1 - Haz_2) ... (1 - Haz_{t-1})
$$
The model models the hazard (chance of event at time $j$ if event has not occurred yet) as below. Time is treated as a categorical variable and each time bin has its own $\alpha$ coefficient
$$
logitHaz(t_j|x_i) = \alpha_j + x_i\beta_i
$$
Model assumes that the hazard function is fixed over time... not sure what that means. I think the ratios stay the same.

You can change the covariates if they change through time. And you can interact with time to get time-dependent effects.

Trying to underweight older data is not trivial. Colleagues also used last year data and discarded the rest. Optimal method is a thing of experimentation, as well as how to update the model.

Event does not have to have happened (for this you have the last bucket, eg >12 months)

Benefits
 - use newer data, can react to novel events (eg Covid)
 - can predict time to event, not "will it happen within 1y"
 - given that you predict the whole survival function you can detect that your model is off much earlier (eg if you see many more events than it predicts in the first month, 2nd month, etc).

(from colleagues)
+ binning
    * ~100bins, up to maximum, equidistant
    * nonparametric
+ would I actually get what I'm looking for: solving the issue of outdated data
    * training set weighting and how old data you go in is an issue now
+ parametric vs nonparametric

## neural network approach

Everything I know so far is that this is a lot more flexible than the Cox approach

## assumptions (with fixes)

These assumptions are true for all the models except Kaplan-Meier.

Non-informative censoring

- check if censored and non-censored observations differ

Survival times are independent

- dependent on study design, can't fix

Hazards are proportional (test with c-log-log plot or Schoenfeld's test)

- stratify on the covariates, ie build different models for different levels (eg a model for male and female)
- time-dependent coefficients, use the interaction btw time and the covariate as the feature

$ln(Haz)$ is a linear function of the covariates (check residual plots)

- typical solutions as in other linear models: transformations, polynomials, binning

Values of the covariates do not change over time

- time dependent covariates

# resources

https://www.youtube.com/watch?v=T_goHnU8Eu4&list=PLqzoL9-eJTNDdnKvep_YHIwk2AMqHhuJ0&index=6