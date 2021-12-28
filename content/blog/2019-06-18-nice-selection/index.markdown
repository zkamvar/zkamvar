---
title: nice selection
author: Zhian N. Kamvar
date: '2019-06-18'
slug: nice-selection
categories:
  - example
  - R
tags:
  - tidyverse
  - tidyselect
  - purrr
  - R
  - functions
  - programming
---

I started writing in R [before the tidyverse became a thing](https://twitter.com/pete_stmarie/status/748195937890115585) 
and I never really had to think about non-standard evaluation when writing 
functions. Those days are long past and I've recently struggled with the
challenge when writing functions for [the R4EPIs project](https://github.com/R4EPI),
which would stick out like ugly little trolls along side tidyverse functions. 

One of my biggest struggles was trying to figure out how, excactly to select a
varaible from a user as either a character string or a bare variable. I had only
known about `rlang::enquo()` and `rlang::sym()`, but I thought I had to use one
or the other.

I had liked `tidyselect::vars_select()` because it gave me characters that I
could use on my data frame columns, and it worked really will with the dots:


```r
f <- function(.df, ...) {
  tidyselect::vars_select(colnames(.df), ...)
}
f(iris, tidyselect::starts_with("Sepal"))
```

```
##   Sepal.Length    Sepal.Width 
## "Sepal.Length"  "Sepal.Width"
```

```r
f(iris, Sepal.Width)
```

```
##   Sepal.Width 
## "Sepal.Width"
```

```r
f(iris, "Sepal.Width")
```

```
##   Sepal.Width 
## "Sepal.Width"
```

But when I would try it with specific arguments, it would scold me if the data
weren't in character form:


```r
f <- function(.df, var) {
  tidyselect::vars_select(colnames(.df), var)
}
f(iris, Sepal.Width)
```

```
## Error: object 'Sepal.Width' not found
```

```r
f(iris, "Sepal.Width")
```

```
## Note: Using an external vector in selections is ambiguous.
## ℹ Use `all_of(var)` instead of `var` to silence this message.
## ℹ See <https://tidyselect.r-lib.org/reference/faq-external-vector.html>.
## This message is displayed once per session.
```

```
##   Sepal.Width 
## "Sepal.Width"
```


I recently found that I could use `!! enquo(var)` to allow the user to input
either a character or a bare variable:


```r
library("rlang")
f <- function(.df, var) {
  tidyselect::vars_select(colnames(.df), !! enquo(var))
}
f(iris, Sepal.Width)
```

```
##   Sepal.Width 
## "Sepal.Width"
```

```r
f(iris, "Sepal.Width")
```

```
##   Sepal.Width 
## "Sepal.Width"
```

And now it finally makes sense!
