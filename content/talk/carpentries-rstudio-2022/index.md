---
title: "Building Accessible Lessons with R and Friends"
excerpt: "how we used our core values to redesign our lesson infrastructure"
date: 2022-07-28T17:50:00Z
show_post_time: false
event: "rstudio::conf(2022)"
event_url: https://rstudio.com/resources/rstudioconf-2022/
author: "Zhian N. Kamvar"
location: "Washington DC, USA"
draft: false
# layout options: single, single-sidebar
layout: single
categories:
- talks
links:
- icon: door-open
  icon_pack: fas
  name: slides
  url: https://bit.ly/BALWRAF
- icon: home
  icon_pack: fas
  name: Documentation
  url: https://carpentries.github.io/workbench/
- icon: r-project
  icon_pack: fab
  name: sandpaper
  url: https://github.com/carpentries/sandpaper/
- icon: r-project
  icon_pack: fab
  name: pegboard
  url: https://github.com/carpentries/pegboard/
- icon: r-project
  icon_pack: fab
  name: varnish
  url: https://github.com/carpentries/varnish/
---

The Carpentries is a global community of volunteers who collaboratively develop
and deliver lessons to build capacity in data and coding skills to researchers
worldwide. In the recent redesign of our lesson infrastructure (serving >100
lessons, used daily by >5K learners), we replaced embedded Jekyll templates
with a workbench of modular and accessible packages using R and Pandoc. By
leveraging renv and knitr for R-based lessons, we provide a seamless and
collaborative lesson development experience that maximizes reproducibility and
minimizes frustration so authors can focus on the contents, not the tooling.
We demonstrate how anyone can use our infrastructure to build customised and
accessible sites for their own lessons or tutorials.
