---
title: The Carpentries Workbench
subtitle: A re-imagining of The Carpentries Lesson Infrastructure that strips away the tooling from the content so lesson authors, maintainers, and contributors can focus on the information and not the formatting.
categories:
- Infrastructure
weight: 10
authors: Zhian N. Kamvar
links:
- icon: door-open
  icon_pack: fas
  name: website
  url: https://carpentries.github.io/workbench
tags:
- Carpentries
- R
---

The Carpentries Workbench is a trio of R packages that deploy, validate, and
style Carpentries-style lessons in a modular way. This design allows lesson
developers, maintainers, and contributors to focus on the content, not the tooling.

## Installation

To install the workbench, you need to have R, git, and pandoc installed. It is
recommended to use RStudio, but it is possible to do so without RStudio.

Once you have these installed, you can install the Workbench packages like so:

```r
install.packages(c("sandpaper", "varnish", "pegboard", "tinkr"),
  repos = c("https://carpentries.r-universe.dev/", getOption("repos")))
```

## Usage

### New Lesson

You can create a new lesson via the templates at https://bit.ly/new-lesson-md
and https://bit.ly/new-lesson-rmd

Once you copy your lesson, move to the next section to build and deploy it
locally.


### Existing Lesson on GitHub

To build your existing lesson locally, you can clone it from GitHub and then
use `sandpaper::serve()` to preview your lesson as you work on it.

```r
usethis::create_from_github("datacarpentry/r-socialsci") # download and open the R for social scientists lesson
sandpaper::serve() # serve the lesson
```
