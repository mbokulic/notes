# designing metrics

**overall evaluation criteria**

Determines if the experiment is successful, ideally one metric since you do not want to do complicated trade-offs.

Two key properties are:

- has to be aligned with company goals (but they are not the same as KPIs!)
- sensitivity (you have to be able to move it)

Designing them is hard. Sometimes you want to move an obvious metric but it is not sensitive. Then you should find correlated metrics that are easier to move.

- start simple
- test by doing obvious things (eg adding too many ads should be negative)
- Setup a *metric evaluation framework*: curate a diverse set of labeled experiments agreed to be positive, negative or neutral with respect to long-term value. Test changes to the OEC on this set

**data quality**

Are the results trustworthy?

- sample ratio mismatch (is the nr of users similar in base and variant?)
- data loss
- click reliability
- cookie churn

**guardrail metrics**: you do not want to ruin these metrics, eg revenue or page loading times. Sometimes small degradation is allowed, a trade-off for the positive outcome

**local feature and diagnostic metrics**: checking to see if you moved what you wanted to move, eg if the users click on the banner that you added. This category will have many metrics.

# mitigating damage

Online experiments are immensely scalable and can hurt the performance of the business. This requires **real-time data** gathering and automating some of the decisions or sending alerts. Here are some of the general ways in which you can minimize the risk of pushing bad code or ideas.

## start small

 - a percentage of your audience, eg 5%
 - partial exposure to the manipulation, eg 1/10 queries if you are testing an online search engine

Since the data will be small, take care that outliers do not dominate your results and aggregate to user level to minimize the impact of extreme users or bots (eg someone who hits your website 100 times an hour).

## detecting interactions
Interactions between your manipulations can happen, and you should mostly be wary of the negative kind. The quintessential example is when one team changes the font to blue, and the other changes the background to the same color. But the accepted wisdom is that interactions are rare.

Interaction prevention

 - run sequential experiments
 - (faster) run non-overlapping experiments: meaning they use the same userbase, ie a user can only appear in one

Measuring interactions is expensive, on the order of $$O(metrics \times experiments^2)$$.

# Increasing metric sensitivity

Not really specific to online experiments, but there are some unique solutions.

Probability to detect a true movement in the metric is equal to the $$p(detect|movement) * p(movement)$$. So it doesn't just depend on statistical power ($$p(detect|movement)$$), but also the sensitivity of the metric.

Increasing statistical power

- increase sample size by running experiments for longer. Does not always help as the metric variance might go up with experiment length (example: number of sessions per user goes up with time). I'm not sure why couldn't you just cap the length of observation per user to say 10 days.
- change the estimand: remove outliers or use non-parametric tests
- use covariate adjustment, eg CUPED (btw, it can also be non-linear, eg use random forest)
  - I'm still not sure why CUPED and not ANCOVA or linear regression. Maybe it takes less processing power. But it can only take 1 variable

Increasing metric sensitivity

- use a different metric
- use a combination of other metrics that correlates with the desired one. I didn't understand this fully

difference between ANCOVA, regression and Cuped

+ Cuped only 1 covariate
+ https://analysis.booking.com/post/71886152.kp?hide_code=1

# metric validation

false positives

+ permutation test instead of bootstrap (shuffle base and variant label)
+ easiest to use an experiment already (or an A/A), practical reasons
+ outcome: you need a uniform distribution
+ K/S test or Chi-square test for goodness of fit (is the distribution uniform?)

for continuous metrics

+ we use Welch test instead of t-test
+ sample mean distribution has to be normal, can use bootstrap to check that
+ both sample mean distribution and p-values have to be distributed as expected
+ check for outliers

# power calculation / sensitivity

heterogeneous effects

- by relevant moderators from previous experiments -> use that to simulate power
- usually not done
- https://medium.com/@astephens.edu/a-b-testing-with-heterogeneous-treatment-7d44cfea8d9e)

teasing out the desired power from stakeholders

 - sensitivity analysis: "we can detect this much in 3 weeks, that much in 4 weeks, etc"
 - "what % are you happy with" is a good phrase
 - bring it to a concrete value (eg 1000 partners will change)

power with covariate adjustment

+ simulation usually (but depends on the assumption that the covariance matrix stays the same)
+ can you use residuals? (doesnt know, maybe I'm wrong)

Can log transform change the sign of the effect size? Not sure yet! Jensen inequality





 - A/A p-value distribution analysis and effect size sensitivity:
    - https://gitlab.booking.com/datascience/segments_decision_intelligence/-/blob/master/hybrid/analyses/202009_t_test_validation/overbooking_analysis.ipynb

# time dependent effects

I need to check this more

# other topics

- Bayesian reasoning
- continuous decision making (ie, making decisions to stop or continue before the end time)
- going beyond average treatment effect, ie diagnosing heterogeneous effects (not sure how different this is from searching for moderators)
- metric development with aggregation: say you have more than one "thing" per unit of measurement and you want to turn it into a binary metric. Say sessions per user, metric is "session longer than 1min".  "all" is less sensitive, and "any" is more sensitive. "any" makes more sense as a manipulation check



# references

[A/B testing at scale tutorial](https://exp-platform.com/2017abtestingtutorial/)

DONE [dirty dozen: 12 common pitfalls in interpreting online experiments](https://exp-platform.com/Documents/2017-08%20KDDMetricInterpretationPitfalls.pdf)

DONE https://exp-platform.com/Documents/2013-02-CUPED-ImprovingSensitivityOfControlledExperiments.pdf

https://www.researchgate.net/profile/Maria-Stone/publication/312211485_Online_Experimentation_Diagnosis_and_Troubleshooting_Beyond_AA_Validation/links/59937b70aca272ec9084ded7/Online-Experimentation-Diagnosis-and-Troubleshooting-Beyond-AA-Validation.pdf?origin=publication_detail

https://www.researchgate.net/profile/Ron-Kohavi/publication/220520287_Unexpected_Results_in_Online_Controlled_Experiments/links/02e7e51bcc03e87038000000/Unexpected-Results-in-Online-Controlled-Experiments.pdf?origin=publication_detail

http://www.robotics.stanford.edu/~ronnyk/2009controlledExperimentsOnTheWebSurvey.pdf

H. Hohnhold, D. O’Brien and D. Tang, “Focusing on the Long-term: It’s Good for Users and Business,” in KDD, 2015

