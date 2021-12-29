---
title: adegenet
subtitle: a R package for the multivariate analysis of genetic markers 
categories:
- R package
weight: 30
authors: Thibaut Jombart, Zhian N. Kamvar (maintainer), et al.
links:
- icon: github
  icon_pack: fab
  name: code
  url: https://github.com/thibautjombart/adegenet/
- icon: r-project
  icon_pack: fab
  name: on CRAN
  url: https://cran.r-project.org/package=adegenet
- icon: question-circle
  icon_pack: far
  name: help forum
  url: https://lists.r-forge.r-project.org/cgi-bin/mailman/listinfo/adegenet-forum
- icon: newspaper
  icon_pack: far
  name: citation
  url: https://doi.org/10.1093/bioinformatics/btn129
- icon: newspaper
  icon_pack: far
  name: citation (tools for SNP data)
  url: https://doi.org/10.1093/bioinformatics/btr521
tags:
- adegenet
- R
- population genetics
---

Adegenet was created by Thibaut Jombart and has been on CRAN since April, 2007.
I was involved in the migration to version 2.0 in the summer of 2015, which 
included consolitading S4 methods and including the `strata()` methods.

I have been maintaining the package by fixing bugs and responding to CRAN
requests since 2020.

## Abstract

Toolset for the exploration of genetic and genomic data. Adegenet provides
formal (S4) classes for storing and handling various genetic data, including
genetic markers with varying ploidy and hierarchical population structure
('genind' class), alleles counts by populations ('genpop'), and genome-wide SNP
data ('genlight'). It also implements original multivariate methods (DAPC,
sPCA), graphics, statistical tests, simulation tools, distance and similarity
measures, and several spatial methods. A range of both empirical and simulated
datasets is also provided to illustrate various methods.

## Installation

You can install adegenet from CRAN in R via

```r
install.packages("adegenet")
```

If you want to use the in-development version of adegenet, you can install it from
my R-universe:

```r
options(repos = c(
    zkamvar = 'https://zkamvar.r-universe.dev',
    CRAN = 'https://cloud.r-project.org'))
install.packages("adegenet")
```

## Getting Help

You can get help for adegenet by registering for the forum at

<https://lists.r-forge.r-project.org/cgi-bin/mailman/listinfo/adegenet-forum>
