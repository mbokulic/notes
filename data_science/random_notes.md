# testing binomial outcomes
ANOVA is a no go, go with logistic with contrasts

http://stats.stackexchange.com/questions/5935/anova-on-binomial-data

# choosing reference for dummy variables
I have not yet found a clear answer on this. As far as I understand, you can do an F-test (ANOVA) to see if any categories are different and if yes, include them. That would mean that at least the min and max value are stat. different.

Other than that, typically you would use the normative category as the reference (eg healthy or control group). If you don't have that, then the largest category. If you don't have an obvious largest category, then use one of the extremes, or even the middle.
http://www.theanalysisfactor.com/strategies-dummy-coding/