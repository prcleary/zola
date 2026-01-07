+++
title = "My R cheatsheet"
+++

> Stuff I find myself Googling for the nth time


### Get name of variable within function

```r
deparse(substitute(x))
```

### Padding numbers with leading zeroes

```r
formatC(1:10, width = 3, flag = '0') 
```

### Remove white space and extraneous characters from text

```r
x <- '  abc <>?|!"£$%^&*()_+}{~@:¬}"  def  123'
gsub('\\W+', '', x)  # Gives "abc_def123"
```

### Set contrasts

```r
options(contrasts = rep('contr.treatment', 2))
```

### Set reference level of factor

```r
thedata[, newvar := relevel(factor(oldvar), 'Reference level here')]
```

