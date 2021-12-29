---
title: poppr
subtitle: R package for population genetic analysis of clonal/sexual organisms
categories:
- R package
weight: 20
authors: Zhian N. Kamvar, et al.
abstract: 
links:
- icon: door-open
  icon_pack: fas
  name: documentation
  url: https://grunwaldlab.github.io/poppr/
- icon: door-open
  icon_pack: fas
  name: tutorial
  url: "https://grunwaldlab.github.io/Population_Genetics_in_R/"
- icon: comment
  icon_pack: far
  name: forum
  url: "https://groups.google.com/d/forum/poppr"
- icon: github
  icon_pack: fab
  name: code
  url: https://github.com/grunwaldlab/poppr/
- icon: r-project
  icon_pack: fab
  name: on CRAN
  url: https://cran.r-project.org/package=poppr
- icon: zenodo
  icon_pack: ai
  name: current doi
  url: https://zenodo.org/record/5486555
- icon: newspaper
  icon_pack: far
  name: citation (PeerJ)
  url: https://doi.org/10.7717/peerj.281
- icon: newspaper
  icon_pack: far
  name: citation (Frontiers)
  url: https://doi.org/10.3389/fgene.2015.00208 
image:
  caption: 'poppr logo'
tags:
- poppr
- R
- population genetics
---

Poppr provides open-source, cross-platform tools for quick analysis of population genetic data enabling focus on data analysis and interpretation. While there are a plethora of packages for population genetic analysis, few are able to offer quick and easy analysis of populations with mixed reproductive modes. Poppr’s main advantage is the ease of use and integration with other packages such as adegenet and vegan, including support for novel methods such as clone correction, multilocus genotype analysis, calculation of Bruvo’s distance, and the index of association. New features in version 2.0 include generation of minimum spanning networks with reticulation, calculation of the index of association for genomic data, and filtering multilocus genotypes based on genetic distance.

## Installation

You can install poppr from CRAN in R via

```r
install.packages("poppr")
```

If you want to use the in-development version of poppr, you can install it from
my R-universe:

```r
options(repos = c(
    zkamvar = 'https://zkamvar.r-universe.dev',
    CRAN = 'https://cloud.r-project.org'))
install.packages("poppr")
```

## Getting Help

You can get help for poppr by visiting our website or searching the google forum
https://groups.google.com/d/forum/poppr


