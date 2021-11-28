
```r
# for comparing floating point numbers
near(1/3, 0.33333333333)

# instead of using two not-equal signs
between(x, -1, 1)
```


```
library(nycflights13)
library(tidyverse)

flights %>%
    # subset the dataset, multiple statements combined with "&"
    filter(month == 11) %>%
    # order the dataset
    arrange(dep_delay)




```


