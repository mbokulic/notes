# housekeeping

## renaming factors
Use character matching with named vectors as in 'vector[c("name1", "name2")]' to create a lookup table for another character  vector. Check Advanced R, p47.

## excluding columns using setdiff

```r
df = data.frame(a=c(1, 2),
                b=c(1, 2),
                c=c(1, 2))
df[, setdiff(names(df), 'c')] # setdiff gives back 'a' and 'b'
```