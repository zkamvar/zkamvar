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


## History

I redesigned and transitioned the underlying build process for more than 50
open source and active lessons, which serve more than 10,000 learners annualy
for The Carpentries volunteer community. This culminated in the creation and
deployment of The Carpentries Workbench, a portable lesson infrastructure,
which separated the content of lessons from the build and styling components
that has significantly improved the quality of life for the more than 100
Lesson Developers and Maintainer volunteers in The Carpentries.

### Products

 * [The Carpentries Workbench](https://carpentries.github.io/workbench) — A
   suite of R packages and GitHub Actions to build, maintain, validate, and
   deploy accessible lessons for data and coding skills.
 * [tinkr](https://docs.ropensci.org/tinkr) — An R package in collaboration
   with rOpenSci that transforms Markdown to XML and back again. This package
   is used for validation of markdown elements without regular expressions and
   is the engine behind the experimental babeldown package, which provides
   automated human language translation of Markdown documents using the DeepL
   API.
 * [Carpentries GitHub Actions](https://github.com/carpentries/actions) — Tools
   for securely provisioning and deploying Carpentries Lessons with arbitrary
   code on GitHub.
 * [Lesson Transition Tool](https://github.com/carpentries/lesson-transition) —
   Automated and flexible transition between Carpentries-style lesson (built
   with Jekyll) to The Carpentries Workbench. This rearranged folder structure,
   updated Markdown syntax, and removed unneeded tooling and generated output
   to reduce the size of the repository and allow the git history to reflect
   authorship.

### Timeline

 * **R&D (2020)** — Research of the fragmented landscape of build strategies
   for lessons and, iterating on feedback from the community, designed an
   initial prototype for the lesson infrastructure with deployment strategies.
   (supervisor: François Michonneau)
 * **Alpha Testing (2021)** — Successful alpha test of prototype, directed
   initial design of website frontend, automated safe and reproducible
   rendering of literate programming in lessons, and implemented first
   iteration of workflow to automate transition of lessons to new
   infrastructure.
   (supervisor: François Michonneau)
 * **Beta Testing (2022)** — Released The Workbench to the community, acheived
   full feature pairity with the former infrastructure, gathered and iterated
   on feedback from early adopters, coordinated and initiated beta testing
   phase with official lesson Maintainers and Instructors, and strategically
   planned to seamlessly transition more than 50 official lessons by Spring
   2023.
   (supervisors: François Michonneau, Kari L. Jordan)
 * **Rollout and Capacity Building (2023)** — Finalized beta testing phase,
   coordinated the seamless and secure rollout of The Workbench to all lessons
   in April–May 2023. Increased capacity within The Core Team by training three
   colleagues in R package development and creating comprehensive [developer
   documentation about The Workbench
   design](https://carpentries.github.io/workbench-dev) and testing practices.
   (supervisors: Toby Hodges, Robert Davey)
