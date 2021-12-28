---
title: Ask developers questions over forums, not emails
author: Zhian N. Kamvar
date: '2019-05-30'
slug: forum-over-email 
categories:
  - r
  - good-practice
tags:
  - email
  - bug
  - forum
  - contact
---

My package, poppr, has been on CRAN for over six years now and it has received
more than 400 citations and just north of 70,000 downloads from the RStudio
cran mirror. I can think of three reasons why this package has been successful:

 1. Our lab gave several workshops using our package over the years
 2. We have written extensive documentation with both [web
    site](https://grunwaldlab.github.io/Population_Genetics_in_R/) and [package
   documentation](https://grunwaldlab.github.io/poppr/).
 3. I actively maintain and answer the vast majority of questions that appear
    on [the *poppr* google group](https://groups.google.com/group/poppr).

I think this last point is crucial. The poppr package has my email address
attached to it in the `Maintainer` field, which is [the person you should
contact if something goes
wrong](http://r-pkgs.had.co.nz/description.html#author), but if anyone sends me
an email about poppr, I will always[^1] send them to the poppr forum (or the
[GitHub issue tracker](https://github.com/grunwaldlab/poppr/issues) if it is a
bug). Even if the user reports [a really embarassing
bug](https://github.com/grunwaldlab/poppr/issues/147), I will do my best to
make sure it [gets fixed and sees the light of day as soon as
possible](https://groups.google.com/d/msg/poppr/ID6H_jYlKwQ/3SjXoHn2CAAJ). I do
this because the best way to build trust with your users is to be brutally
honest in the most transparent way possible; the best way I know to do this is
through an official forum.

For users, workshops and documentation are nice introductions, but a forum is
really one of the few places where you can actually engage with the developers
and the community to get into nitty-gritty details of an analysis, get help
with some unclear documentation, or report some strange behavior that may or
may not be a bug. From a developer perspective, a forum is an invaluable tool
that allows you to see how your users work and shows you a world of analysis
beyond what's inside your head or in your immediate workgroup. I've had people
ask about [clever workarounds for violated
assumptions](https://groups.google.com/d/msg/poppr/86p9M6mbqDk/9bYMotlwiQcJ),
report [bugs](https://groups.google.com/d/msg/poppr/81rfOsyItj0/uc82jz26AQAJ),
and [request
features](https://groups.google.com/d/msg/poppr/K8Z9HjEAvJ4/I82_6MhVy3IJ) that
eventually became part of poppr. The best part of all of this is that it is all
public, which means that it's possible for users to find answers to their
questions before they even have to ask it.

[^1]: There are some exceptions where the user may be unable to sign up to the
      forum or they have sensitive data they do not wish to share, but these are
      exceedingly rare.

