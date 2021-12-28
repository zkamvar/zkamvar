---
title: Recreating Figure 3
author: Zhian N. Kamvar
date: '2016-02-10'
slug: recreating-figure-3
categories:
  - R
  - example
tags:
  - poppr
  - multilocus genotype
  - ggplot2
banner: img/banners/fig3.png
---




Motivation
==========

In February of 2016, I got an email asking if I could provide the code to
recreate [figure three][fig3] from my [article in Phytopathology][sod] on the
outbreak of *Phytophthora ramorum* in Curry County OR from 2001 to 2014
(paywalled, but [you can find a copy here][sod-free]).

While I have the [code used for the analysis on github][analysis], it's a lot of
stuff to sort through, considering that it was my first foray in attempting a
reproducible analysis, so for this post, I'm going to recreate it using current
tools.

I created figure three originally in two parts with ggplot2 and then manually
aligned the two figures in inkscape. Since then, the package cowplot has come
around and made this process easier. I have my old code up here:
[mlg_distribution.Rmd][mlgdist], and since the packages have changed since then,
I'm redoing the code here.

Analysis
========

## Loading Packages/Data


```r
library("poppr")    # Note, v.2.2.0 or greater is needed for the %>% operator
```

```
## Loading required package: adegenet
```

```
## Loading required package: ade4
```

```
## 
##    /// adegenet 2.1.5 is loaded ////////////
## 
##    > overview: '?adegenet'
##    > tutorials/doc/questions: 'adegenetWeb()' 
##    > bug reports/feature requests: adegenetIssues()
```

```
## Registered S3 method overwritten by 'pegas':
##   method      from
##   print.amova ade4
```

```
## This is poppr version 2.9.3. To get started, type package?poppr
## OMP parallel support: unavailable
```

```r
library("ggplot2")  # Plotting
library("cowplot")  # Grouping the plots
```

The data from the paper has been stored in *poppr* as "Pram", but it includes 
nursery data. I'm removing it here.


```r
data("Pram")
mll(Pram) <- "original"
Pram
```

```
## 
## This is a genclone object
## -------------------------
## Genotype information:
## 
##     98 original multilocus genotypes 
##    729 diploid individuals
##      5 codominant loci
## 
## Population information:
## 
##      3 strata - SOURCE, YEAR, STATE
##      9 populations defined - 
## Nursery_CA, Nursery_OR, JHallCr_OR, ..., Winchuck_OR, ChetcoMain_OR, PistolRSF_OR
```

```r
ramdat <- Pram %>%
  setPop(~SOURCE) %>%               # Set population strata to SOURCE (forest/nursery)
  popsub(blacklist = "Nursery") %>% # remove the nursery derived samples
  setPop(~YEAR)                     # Set the strata to YEAR of epidemic
```

```
## Warning in popsub(., blacklist = "Nursery"): the option blacklist is deprecated as of poppr version 2.8.7. Please use `exclude` in the future 
## 
## Please use this as a replacement:
##   popsub(gid = ., exclude = "Nursery")
```

```r
# A color palette (unnecessary)
ncolors <- max(mll(ramdat))
myPal   <- setNames(funky(ncolors), paste0("MLG.", seq(ncolors)))
```

Creating the Barplot
--------------------

The barplot is a barplot of the MLG counts ordered from most abundant to least
abundant.


```r
# This obtains a table of sorted MLG counts for adjusting the axes.
mlg_order <- table(mll(ramdat)) %>% 
  sort() %>% 
  data.frame(MLG = paste0("MLG.", names(.)), Count = unclass(.))

# Creating the bar plot
bars <- ggplot(mlg_order, aes(x = MLG, y = Count, fill = MLG)) + 
  geom_bar(stat = "identity") +
  theme_classic() +
  scale_y_continuous(expand = c(0, 0), limits = c(0, 180)) +
  scale_fill_manual(values = myPal) +
  geom_text(aes(label = Count), size = 2.5, hjust = 0, fontface = "bold") +
  theme(axis.text.y = element_blank()) + 
  theme(axis.ticks.y = element_blank()) +
  theme(legend.position = "none") +
  theme(text = element_text(family = "Helvetica")) +
  theme(axis.title.y = element_blank()) +
  # From the documentation for theme: top, right, bottom, left
  theme(plot.margin = unit(c(1, 1, 1, 0), "lines")) + 
  scale_x_discrete(limits = mlg_order$MLG) +
  coord_flip()

bars
```

<img src="{{< blogdown/postref >}}index_files/figure-html/barplot-1.png" width="50%" style="display: block; margin: auto;" />

Creating the Subway plot
------------------------

This plot displays the MLGs occurring across years. It's a nice graphical way of
displaying the results of `mlg.crosspop()` when the populations are years.


```r
mlg_range <- mlg.crosspop(ramdat, mlgsub = unique(mll(ramdat)), 
                          df = TRUE, quiet = TRUE)
names(mlg_range)[2] <- "Year"

# Creating the subway plot
ranges <- ggplot(mlg_range, aes(x = Year, y = MLG, group = MLG, color = MLG)) + 
  geom_line(size = 1, linetype = 1) + 
  geom_point(size = 5, pch = 21, fill = "white") +
  geom_text(aes(label = Count), color = "black", size = 2.5) + 
  scale_color_manual(values = myPal) + 
  ylab("Multilocus Genotype") +
  theme_bw() + 
  theme(axis.text.x = element_text(angle = 90, hjust = 1, vjust = 0.5)) +
  theme(text = element_text(family = "Helvetica")) +
  theme(legend.position = "none") +
  theme(axis.line = element_line(colour = "black")) +
  # From the documentation for theme: top, right, bottom, left
  theme(plot.margin = unit(c(1, 0, 1, 1), "lines")) +
  scale_y_discrete(limits = mlg_order$MLG)

ranges
```

<img src="{{< blogdown/postref >}}index_files/figure-html/subwayplot-1.png" width="50%" style="display: block; margin: auto;" />

> **A word on margins**
> 
> Cowplot is nice for placing the ggplot objects next to each other in one
> frame, but it likes to give them room to spread out. To get the plots as close
> together as possible, I'm cutting out the left and right margins of the
> barplot and subway plot, respectively. This is done with the `plot.margin`
> argument to `theme()` which organizes the widths as **top**, **right**,
> **bottom**, **left**.


Aligning with cowplot
---------------------

Cowplot's `plot_grid()` will fit these two plots together. Originally, I had to 
export these plots and align them by hand in inkscape, but now, they can be 
plotted together and aligned in one swoop. There's some fiddling to be done with
the margins, but it might be easier to export it as an svg, and then slide one
over to the other in 2 minutes in inkscape.



```r
cowplot::plot_grid(ranges, bars, align = "h", rel_widths = c(2.5, 1))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/cowplot-1.png" width="50%" style="display: block; margin: auto;" />

Conclusion
==========

This plot was done when I was originally toying with the idea of keeping my
analysis open. Of course, I know more things now than I did then, but I do enjoy
the fact that I can go back a year later and recreate the exact plot from start
to finish.

Session Information
===================


```r
options(width = 100)
devtools::session_info()
```

```
## ─ Session info ─────────────────────────────────────────────────────────────────────────
##  setting  value
##  version  R version 4.1.2 (2021-11-01)
##  os       macOS Catalina 10.15.7
##  system   x86_64, darwin17.0
##  ui       X11
##  language (EN)
##  collate  en_US.UTF-8
##  ctype    en_US.UTF-8
##  tz       America/Los_Angeles
##  date     2021-12-27
##  pandoc   2.12 @ /Users/zhian/anaconda3/bin/ (via rmarkdown)
## 
## ─ Packages ─────────────────────────────────────────────────────────────────────────────
##  package     * version date (UTC) lib source
##  ade4        * 1.7-18  2021-09-16 [1] CRAN (R 4.1.0)
##  adegenet    * 2.1.5   2021-10-09 [1] https://zkamvar.r-universe.dev (R 4.1.1)
##  ape           5.5     2021-04-25 [1] CRAN (R 4.1.0)
##  assertthat    0.2.1   2019-03-21 [1] CRAN (R 4.1.0)
##  blogdown      1.7     2021-12-19 [1] CRAN (R 4.1.0)
##  bookdown      0.24    2021-09-02 [1] CRAN (R 4.1.0)
##  boot          1.3-28  2021-05-03 [2] CRAN (R 4.1.2)
##  bslib         0.3.1   2021-10-06 [1] CRAN (R 4.1.1)
##  cachem        1.0.6   2021-08-19 [1] CRAN (R 4.1.0)
##  callr         3.7.0   2021-04-20 [1] CRAN (R 4.1.0)
##  cli           3.1.0   2021-10-27 [1] CRAN (R 4.1.0)
##  cluster       2.1.2   2021-04-17 [2] CRAN (R 4.1.2)
##  codetools     0.2-18  2020-11-04 [2] CRAN (R 4.1.2)
##  colorspace    2.0-2   2021-06-24 [1] CRAN (R 4.1.0)
##  cowplot     * 1.1.1   2020-12-30 [1] CRAN (R 4.1.0)
##  crayon        1.4.2   2021-10-29 [1] CRAN (R 4.1.0)
##  DBI           1.1.1   2021-01-15 [1] CRAN (R 4.1.0)
##  desc          1.4.0   2021-09-28 [1] CRAN (R 4.1.1)
##  devtools      2.4.3   2021-11-30 [1] CRAN (R 4.1.0)
##  digest        0.6.29  2021-12-01 [1] CRAN (R 4.1.0)
##  dplyr         1.0.7   2021-06-18 [1] CRAN (R 4.1.0)
##  ellipsis      0.3.2   2021-04-29 [1] CRAN (R 4.1.0)
##  evaluate      0.14    2019-05-28 [1] CRAN (R 4.1.0)
##  fansi         0.5.0   2021-05-25 [1] CRAN (R 4.1.0)
##  farver        2.1.0   2021-02-28 [1] CRAN (R 4.1.0)
##  fastmap       1.1.0   2021-01-25 [1] CRAN (R 4.1.0)
##  fs            1.5.2   2021-12-08 [1] CRAN (R 4.1.0)
##  generics      0.1.1   2021-10-25 [1] CRAN (R 4.1.0)
##  ggplot2     * 3.3.5   2021-06-25 [1] CRAN (R 4.1.0)
##  glue          1.6.0   2021-12-17 [1] CRAN (R 4.1.0)
##  gtable        0.3.0   2019-03-25 [1] CRAN (R 4.1.0)
##  highr         0.9     2021-04-16 [1] CRAN (R 4.1.0)
##  htmltools     0.5.2   2021-08-25 [1] CRAN (R 4.1.0)
##  httpuv        1.6.4   2021-12-15 [1] https://carpentries.r-universe.dev (R 4.1.2)
##  igraph        1.2.10  2021-12-15 [1] CRAN (R 4.1.0)
##  jquerylib     0.1.4   2021-04-26 [1] CRAN (R 4.1.0)
##  jsonlite      1.7.2   2020-12-09 [1] CRAN (R 4.1.0)
##  knitr         1.37    2021-12-16 [1] CRAN (R 4.1.0)
##  labeling      0.4.2   2020-10-20 [1] CRAN (R 4.1.0)
##  later         1.3.0   2021-08-18 [1] CRAN (R 4.1.0)
##  lattice       0.20-45 2021-09-22 [2] CRAN (R 4.1.2)
##  lifecycle     1.0.1   2021-09-24 [1] CRAN (R 4.1.0)
##  magrittr      2.0.1   2020-11-17 [1] CRAN (R 4.1.0)
##  MASS          7.3-54  2021-05-03 [1] CRAN (R 4.1.0)
##  Matrix        1.4-0   2021-12-08 [2] CRAN (R 4.1.0)
##  memoise       2.0.1   2021-11-26 [1] CRAN (R 4.1.0)
##  mgcv          1.8-38  2021-10-06 [1] CRAN (R 4.1.1)
##  mime          0.12    2021-09-28 [1] CRAN (R 4.1.0)
##  munsell       0.5.0   2018-06-12 [1] CRAN (R 4.1.0)
##  nlme          3.1-153 2021-09-07 [2] CRAN (R 4.1.2)
##  pegas         1.1     2021-12-16 [1] CRAN (R 4.1.0)
##  permute       0.9-5   2019-03-12 [1] CRAN (R 4.1.0)
##  pillar        1.6.4   2021-10-18 [1] CRAN (R 4.1.0)
##  pkgbuild      1.3.0   2021-12-09 [1] CRAN (R 4.1.0)
##  pkgconfig     2.0.3   2019-09-22 [1] CRAN (R 4.1.0)
##  pkgload       1.2.4   2021-11-30 [1] CRAN (R 4.1.0)
##  plyr          1.8.6   2020-03-03 [1] CRAN (R 4.1.0)
##  polysat       1.7-6   2021-12-08 [1] CRAN (R 4.1.0)
##  poppr       * 2.9.3   2021-09-07 [1] CRAN (R 4.1.1)
##  prettyunits   1.1.1   2020-01-24 [1] CRAN (R 4.1.0)
##  processx      3.5.2   2021-04-30 [1] CRAN (R 4.1.0)
##  promises      1.2.0.1 2021-02-11 [1] CRAN (R 4.1.0)
##  ps            1.6.0   2021-02-28 [1] CRAN (R 4.1.0)
##  purrr         0.3.4   2020-04-17 [1] CRAN (R 4.1.0)
##  R6            2.5.1   2021-08-19 [1] CRAN (R 4.1.0)
##  raster        3.5-9   2021-12-10 [1] CRAN (R 4.1.0)
##  Rcpp          1.0.7   2021-07-07 [1] CRAN (R 4.1.0)
##  remotes       2.4.2   2021-11-30 [1] CRAN (R 4.1.0)
##  reshape2      1.4.4   2020-04-09 [1] CRAN (R 4.1.0)
##  rlang         0.4.12  2021-10-18 [1] CRAN (R 4.1.0)
##  rmarkdown     2.11    2021-09-14 [1] CRAN (R 4.1.0)
##  rprojroot     2.0.2   2020-11-15 [1] CRAN (R 4.1.0)
##  sass          0.4.0   2021-05-12 [1] CRAN (R 4.1.0)
##  scales        1.1.1   2020-05-11 [1] CRAN (R 4.1.0)
##  seqinr        4.2-8   2021-06-09 [1] CRAN (R 4.1.0)
##  sessioninfo   1.2.2   2021-12-06 [1] CRAN (R 4.1.0)
##  shiny         1.7.1   2021-10-02 [1] CRAN (R 4.1.0)
##  sp            1.4-6   2021-11-14 [1] CRAN (R 4.1.0)
##  stringi       1.7.6   2021-12-04 [1] https://carpentries.r-universe.dev (R 4.1.2)
##  stringr       1.4.0   2019-02-10 [1] CRAN (R 4.1.0)
##  terra         1.4-22  2021-11-24 [1] CRAN (R 4.1.0)
##  testthat      3.1.1   2021-12-03 [1] CRAN (R 4.1.0)
##  tibble        3.1.6   2021-11-07 [1] CRAN (R 4.1.0)
##  tidyselect    1.1.1   2021-04-30 [1] CRAN (R 4.1.0)
##  usethis       2.1.5   2021-12-09 [1] CRAN (R 4.1.0)
##  utf8          1.2.2   2021-07-24 [1] CRAN (R 4.1.0)
##  vctrs         0.3.8   2021-04-29 [1] CRAN (R 4.1.0)
##  vegan         2.5-7   2020-11-28 [1] CRAN (R 4.1.0)
##  withr         2.4.3   2021-11-30 [1] CRAN (R 4.1.0)
##  xfun          0.29    2021-12-14 [1] CRAN (R 4.1.0)
##  xtable        1.8-4   2019-04-21 [1] CRAN (R 4.1.0)
##  yaml          2.2.1   2020-02-01 [1] CRAN (R 4.1.0)
## 
##  [1] /Users/zhian/R
##  [2] /Library/Frameworks/R.framework/Versions/4.1/Resources/library
## 
## ────────────────────────────────────────────────────────────────────────────────────────
```
[fig3]: https://www.researchgate.net/publication/278039693_Spatial_and_Temporal_Analysis_of_Populations_of_the_Sudden_Oak_Death_Pathogen_in_Oregon_Forests/figures
[sod]: http://apsjournals.apsnet.org/doi/10.1094/PHYTO-12-14-0350-FI
[sod-free]: https://www.researchgate.net/publication/278039693_Spatial_and_Temporal_Analysis_of_Populations_of_the_Sudden_Oak_Death_Pathogen_in_Oregon_Forests
[analysis]: https://github.com/zkamvar/Sudden_Oak_Death_in_Oregon_Forests#readme
[mlgdist]: https://github.com/zkamvar/Sudden_Oak_Death_in_Oregon_Forests/blob/master/mlg_distribution.Rmd
