---
title: Auto-building drat with gh actions
author: Zhian N. Kamvar
date: '2020-01-14'
slug: gh-drat
categories:
  - github
  - git
  - package
  - R
  - drat
  - example
tags:
  - Continuous Integration
  - open source
  - packages
  - R4EPIs
  - R
  - RECON
subtitle: ''
summary: 'Combining GitHub actions and {drat.builder} makes it easy to create an auto-building drat repository for your R packages'
authors: ["Zhian N. Kamvar"]
lastmod: '2020-01-14T13:00:30Z'
featured: true 
image:
  caption: 'screen capture of GitHub Action build process showing a successfully updated drat repository'
  focal_point: ''
  preview_only: no
projects: []
---

> tl;dr: I created an auto-building drat repository at https://github.com/r4epi/drat that uses GitHub actions: <https://github.com/R4EPI/drat/blob/master/.github/workflows/update-pkgs.yaml>. It checks for updates and rebuilds packages every week. 

CRAN is a wonderful service. If you have a package hosted there, then you have the benefit of having your package continuously checked against several versions of R in the current package ecosystem. Moreover, the installation of packages *just works*™ with `install.packages()`. The only drawback is that the barrier for entry can be pretty high and, even if you have a package on CRAN already, it can sometimes take days for any important bugfixes to be updated there. One of thealternatives/backups is to host a package on GitHub and have users install it with `remotes::install_github()`. This is sub-optimal because your packages may be scattered around different organizations and your users don't really need to think about what username the GitHub repo belongs to. The other alternative is a {drat} repository, which takes a bit more setting up, but is worth it.

In this post, I will talk about how I set up a {drat} repository to automatically update my packages on a weekly basis using GitHub Actions and the [{drat.builder}](https://github.com/richfitz/drat.builder#readme) package. 

# Babby's first drat repo

## What's drat?

For the uninitiated, a [{drat}](http://dirk.eddelbuettel.com/code/drat.html) is a package that allows people to host CRAN-like repositories on GitHub leveraging two facts:

 1. GitHub provides free websites for each repository under
    `https://[USER].github.io/[REPO]`.
 2. A website can be a valid R repository as long as it has the source packages
    and package index under `[URL]/src/contrib/`

From there, any user can use {drat} to provide a path to installing your packages. It literally takes two commands to get people up-and-running:

```r
drat:::add("R4EPI")
install.packages("epibuffet")
```

## Simply done

Up until now, I've been avoiding setting up a {drat} repository for my projects because I had been using `remotes::install_github()` for so long, it just felt more natural^[Along with the fact that you as a maintainer had to do literally nothing to keep it up-to-date (as long as you had CI running, ofc)]. I recently found out that Rich FitzJohn had created a package a few years ago called [{drat.builder}](https://github.com/richfitz/drat.builder#readme) that allows you to simply specify a file called `packages.txt` with a list of GitHub sources and it would take care of the rest.

My goal was to set up a repository for the R4EPIs project (<https://r4epis.netlify.com>), because we were finding that users were getting a lot of errors installing from GitHub (largely related to packages like {curl} updating while remotes had it loaded) and wanted a cleaner way for them to install the necessary packages. In the end, it took three steps:

1. create a drat repo on GitHub (<https://github.com/R4EPI/drat>) and clone it
   to my computer
2. add a file called `packages.txt`
3. run `drat.builder::build()`

The package created a commit for each package it built and all I had to do was push the changes and enable GitHub pages for the repo. After that, users were able to install the [{sitrep}](https://R4EPI.github.io/sitrep) template package with two commands.

# GitHub in the Action

## Travis-ty

Now that I had a drat repository for R4EPIs, I wanted to make sure that it would stay up-to-date without my help. The {drat} package already has [instructions for using Travis CI to update a drat repository](https://cran.r-project.org/web/packages/drat/vignettes/CombiningDratAndTravis.html), but this is a script that you add to your package repository to send to your drat repository, which means that you have to add it to each and every package repository you want to send to drat. I did not want to have to add a secret github token to every repository that I wanted to be included in the DRAT repo. 

## 28 commits later

Luckliy, GitHub recently came out with [GitHub Actions](https://github.com/features/actions), and the RStudio team came out with [r-lib/actions](https://github.com/r-lib/actions). I decided to test things out by creating a repository (<https://github.com/zkamvar/drat>) with only a `packages.txt` and generating the packages through GitHub Actions. 

It took me 28 tries to get things working correctly. Here are some things that I found out along the way:

1. [Working on windows is hard.](https://github.com/zkamvar/drat/commit/be17f687999aa89767d9ea57db077308677cd81b) `Rscript -e "install.packages('remotes')"` works but `Rscript -e 'install.packages("remotes")'` fails 😦
2. Working on windows is hard part deux. [You apparently can't call git from inside R.](https://github.com/zkamvar/drat/commit/c1e235cf9c05c9e8d443ec5b4af260a34196d0e9)
3. Pull requests will build and push to the branch of the PR <https://github.com/zkamvar/drat/pull/1>

In the end, I had a yaml file under 30 lines that will update the repository every week (<https://github.com/R4EPI/drat/blob/master/.github/workflows/update-pkgs.yaml>). Not only that, but because it's all controlled via the packages text file, I only need to have access to the internet to update the package list. For posterity, here's the full file:

```
on:
  schedule:
    - cron: "0 12 * * WED"
  push:
    path:
      packages.txt
      .github/workflows/render-pkgs.yaml

name: Build packages

jobs:
  render:
    name: Build packages on Ubuntu
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - uses: r-lib/actions/setup-r@v1
      - uses: r-lib/actions/setup-pandoc@v1
      - name: Install remotes
        run: Rscript -e "install.packages('remotes')"
      - name: Install drat.builder 
        run: Rscript -e "remotes::install_github('richfitz/drat.builder')"
      - name: Update DRAT
        run: |
          git config --global user.name "[drat bot]"
          Rscript -e "drat.builder::build()"
      - name: Push results
        run: |
          git push https://${{github.actor}}:${{secrets.GITHUB_TOKEN}}@github.com/${{github.repository}}.git HEAD:${{ github.ref }} || echo "No changes to commit"
```

# Further steps

I'm really glad that I got this figured out, but there are still a few steps to
take if I want things to be seamless for users. Namely, I want to figure out
how to build binary packages. GitHub offers windows and macOS machines, but the
{drat.builder} package doesn't build binaries. It would be good to re-engineer
the package into typescript and add the feature to build the binaries as well.
That way new packages and bug fixes are easier to deliver to users.

All in all, I'm really thankful for the R community for turning something that would have been a week-long project into a half-day project. 
