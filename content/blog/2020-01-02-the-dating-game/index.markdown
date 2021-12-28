---
title: The dating game
author: Zhian N. Kamvar
date: '2020-01-02'
slug: [the-dating-game]
categories:
  - R
tags:
  - packages
  - RECON
  - R
  - programming
  - dates
  - text
---

# Dates are not so sweet

It is known: parsing dates entered by humans is a huge pain:

![ISO 8601 from XKCD](https://imgs.xkcd.com/comics/iso_8601.png)  
Source: https://xkcd.com/1179/

{{% tweet 1212501345720250368 %}}

Dates are a never-ending source of fresh hell. There are so many different ways
parsing dates can go wrong. It’s not quite as bad as [the horror that is
parsing HTML with regex](https://stackoverflow.com/a/1732454/2752888), but it’s
close. For example, consider [the regex used to parse valid ISO 8601 dates](https://www.myintervals.com/blog/2009/05/20/iso-8601-date-validation-that-doesnt-suck/)…
you know, the dates that are supposed to be “the good ones”:

    ^([\+-]?\d{4}(?!\d{2}\b))((-?)((0[1-9]|1[0-2])(\3([12]\d|0[1-9]|3[01]))?|W([0-4]\d|5[0-2])(-?[1-7])?|(00[1-9]|0[1-9]\d|[12]\d{2}|3([0-5]\d|6[1-6])))([T\s]((([01]\d|2[0-3])((:?)[0-5]\d)?|24\:?00)([\.,]\d+(?!:))?)?(\17[0-5]\d([\.,]\d+)?)?([zZ]|([\+-])([01]\d|2[0-3]):?([0-5]\d)?)?)?)?$

Oh deer lord 😱

This is only just the beginning. There are *so many* blog posts already dedicated
to working with dates in R, and I’m not going to re-hash the whole rigormarole of
explaining the difference between Date and POSIXt classes. I’m just going to
point you to some excellent walkthroughs such as [this gist by Bonnie Dixon (hosted
by Noam Ross)](https://www.noamross.net/archives/2014-02-10-using-times-and-dates-in-r-presentation-code/).

What I want to focus on is: what tools are available to parse date strings to
the `Date` class in R and how well do they work to weird situations? Some of the
common things I’ve seen in my work for example:

-   an ambiguous mix of dd/mm/yy and mm/dd/yy format (e.g. 02/04/20 or 04/02/20)
-   Dates are in French, but you’re working in an English locale (e.g. 04 Février 2020)
-   Dates are imported from Excel (e.g. 43865, which are represented as the either the number of days since 1899-12-30 if you are importing from Windows OR the number of days since 1904-01-01 on MacOS)[^1]

# There’s a package for that

I’m immediately familiar with three packages on CRAN that are solely[^2] dedicated to parsing
dates: [{lubridate}](https://lubridate.tidyverse.org/),
[{anytime}](http://dirk.eddelbuettel.com/code/anytime.html),
[{parsedate}](https://cran.r-project.org/package=parsedate). However, I know that
I’m probably missing some, so I’ll try to use the
[{pkgsearch}](https://cran.r-project.org/package=pkgsearch) package to find them.
I know that I want to search for any package that mentions “date” in the title
and has either “parse,” “handle,” “convert,” or “detect” in the description.

> N.B. I *was* missing a package as Jim Hester kindly pointed out:
>
> {{% tweet 1212787019677609985 %}}

``` r
library("dplyr")
library("stringr")
library("pkgsearch")

date_pkgs <- pkg_search("date", size = 200) %>%
  filter(str_detect(title,       "[Dd]ate") & 
         str_detect(description, "[Pp]ars|[Hh]andl|[Cc]onvert|[Dd]etect")) %>%
  filter(str_detect(maintainer_name, "Zhian", negate = TRUE)) %>% # my packages don't parse dates
  filter(package != "chron") %>% # chron is a well-known package that doesn't do text to date conversions
  arrange(desc(downloads_last_month))

date_pkgs[] %>%
  mutate(package = str_glue("[{package}](https://cran.r-project.org/package={package})")) %>%
  select(Package = package, `Downloads Last Month` = downloads_last_month, Title = title) %>%
  knitr::kable(format.args = list(big.mark = ","))
```

| Package                                                           | Downloads Last Month | Title                                                            |
|:------------------------------------------------------------------|---------------------:|:-----------------------------------------------------------------|
| [lubridate](https://cran.r-project.org/package=lubridate)         |              859,067 | Make Dealing with Dates a Little Easier                          |
| [anytime](https://cran.r-project.org/package=anytime)             |               78,442 | Anything to ‘POSIXct’ or ‘Date’ Converter                        |
| [parsedate](https://cran.r-project.org/package=parsedate)         |               19,323 | Recognize and Parse Dates in Various Formats, Including All ISO  |
| 8601 Formats                                                      |                      |                                                                  |
| [datetime](https://cran.r-project.org/package=datetime)           |                7,047 | Nominal Dates, Times, and Durations                              |
| [date](https://cran.r-project.org/package=date)                   |                4,351 | Functions for Handling Dates                                     |
| [incidence](https://cran.r-project.org/package=incidence)         |                3,271 | Compute, Handle, Plot and Model Incidence of Dated Events        |
| [clock](https://cran.r-project.org/package=clock)                 |                2,389 | Date-Time Types and Tools                                        |
| [incidence2](https://cran.r-project.org/package=incidence2)       |                1,071 | Compute, Handle and Plot Incidence of Dated Events               |
| [MMWRweek](https://cran.r-project.org/package=MMWRweek)           |                  828 | Convert Dates to MMWR Day, Week, and Year                        |
| [dint](https://cran.r-project.org/package=dint)                   |                  735 | A Toolkit for Year-Quarter, Year-Month and Year-Isoweek Dates    |
| [datetimeutils](https://cran.r-project.org/package=datetimeutils) |                  695 | Utilities for Dates and Times                                    |
| [datefixR](https://cran.r-project.org/package=datefixR)           |                  389 | Fix Really Messy Dates                                           |
| [rccdates](https://cran.r-project.org/package=rccdates)           |                  340 | Date Functions for Swedish Cancer Data                           |
| [jalcal](https://cran.r-project.org/package=jalcal)               |                  310 | Conversion Between Jalali (Persian or Solar Hijri) and Gregorian |
| Calendar Dates                                                    |                      |                                                                  |
| [unstruwwel](https://cran.r-project.org/package=unstruwwel)       |                  229 | Detect and Parse Historic Dates                                  |

There are a few more, but then there are some that don’t really parse dates,
such as {dint}, {MMWRweek}, and {datetime}. This leaves us with a total of seven
packages on CRAN that handle dates: {lubridate}, {anytime}, {parsedate},
{date}, {datetimeutils}, and {rccdates}.

# Thunderdate

Let’s see how these packages do on our date gauntlet, specified from above.

``` r
the_dates   <- c("2020-02-04", "04 February 2020", "2/4/20"  , "4/2/20"  , "04 Février 2020", 43865)
# Formats for {base} R need to be in the exact order
the_formats <- c("%Y-%m-%d"  , "%d %B %Y"        , "%m/%d/%y", "%d/%m/%y", "%d %B %Y")
# Formats for {lubridate} are much easier to read
print(lub_formats <- unique(gsub("[[:punct:][:space:]]", "", the_formats)))
```

    ## [1] "Ymd" "dBY" "mdy" "dmy"

Setting up the {readr} function for use with {purrr}

``` r
the_locales <- c("en", "en", "en", "en", "fr", "en")
readr_parse_date <- function(date, format, locale) {
  readr::parse_date(date, format = format, locale = readr::locale(locale))
}
purrrlist <- list(the_dates, c(the_formats, NA), the_locales)
the_origin <- as.Date("1970-01-01")
```

``` r
res <- tibble(
  original      = the_dates,
  base          = as.Date(the_dates,                    format = the_formats),
  lubridate     = lubridate::parse_date_time(the_dates, orders = lub_formats),
  readr         = purrr::pmap_dbl(purrrlist, readr_parse_date) %>% as.Date(origin = the_origin),
  anytime       = anytime::anydate(the_dates),
  parsedate     = parsedate::parse_date(the_dates),
  date          = date::as.date(the_dates),
  datetimeutils = datetimeutils::guess_datetime(the_dates),
  rccdates      = rccdates::as.Dates(the_dates)
) %>%
  mutate_at(-1, as.Date) 
```

    ## Warning: 1 failed to parse.

    ## Warning: 1 parsing failure.
    ## row col     expected actual
    ##   1  -- date like NA  43865

| original         | base       | lubridate  | readr      | anytime    | parsedate  | date       | datetimeutils | rccdates   |
|:-----------------|:-----------|:-----------|:-----------|:-----------|:-----------|:-----------|:--------------|:-----------|
| 2020-02-04       | 2020-02-04 | 2020-02-04 | 2020-02-04 | 2020-02-04 | 2020-02-04 | NA         | NA            | 2020-02-04 |
| 04 February 2020 | 2020-02-04 | 2020-04-20 | 2020-02-04 | 2020-02-04 | 2020-02-04 | 2020-02-04 | NA            | NA         |
| 2/4/20           | 2020-02-04 | 2020-02-04 | 2020-02-04 | NA         | 2020-02-04 | 1920-02-04 | NA            | NA         |
| 4/2/20           | 2020-02-04 | 2020-04-02 | 2020-02-04 | NA         | 2020-04-02 | 1920-04-02 | NA            | NA         |
| 04 Février 2020  | NA         | 2020-04-20 | 2020-02-04 | NA         | 2020-01-04 | NA         | NA            | NA         |
| 43865            | NA         | NA         | NA         | 4386-01-01 | 2021-12-27 | NA         | NA            | NA         |

Table 1: Results

| base | lubridate | readr | anytime | parsedate | date | datetimeutils | rccdates |
|-----:|----------:|------:|--------:|----------:|-----:|--------------:|---------:|
|    4 |         2 |     5 |       2 |         3 |    1 |             0 |        1 |

Table 2: Dates correctly parsed (of 6)

There’s a lot going on here, so I’ll summarize a few things:

1.  None of these format perfectly, locales and numeric dates are hard to handle.
2.  {datetimeutils} could not convert a single date presented and {rccdate} could
    only parse the ISO 8601 formatted date (but it’s not meant to be a general
    date parsing package).
3.  By far, the most successful package was {readr}, but it takes a bit of work
    to set up the function to return a date vector with multiple locales and
    formats.
4.  The most successful “magic” package (one without knowledge of formats) was
    {parsedate}.
5.  {anytime} is super conservative, but will assume that a string of numbers
    represents a date (e.g. 20200204).
6.  {lubridate} does fairly well, but it makes some strange mistakes with the
    month spelled out.
7.  {base} only does well here because I gave it the EXACT specifications for
    each date. If I didn’t, it would only be able to parse the ISO 8601 date.

For those curious, this is what the results would look like from a French locale:

``` r
library("withr")
with_locale(c(LC_TIME = "fr_FR.UTF-8"), {
  res <- tibble(
    original      = the_dates,
    base          = as.Date(the_dates,                    format = the_formats),
    lubridate     = lubridate::parse_date_time(the_dates, orders = lub_formats),
    readr         = purrr::pmap_dbl(purrrlist, readr_parse_date) %>% as.Date(origin = the_origin),
    anytime       = anytime::anydate(the_dates),
    parsedate     = parsedate::parse_date(the_dates),
    date          = date::as.date(the_dates),
    datetimeutils = datetimeutils::guess_datetime(the_dates),
    rccdates      = rccdates::as.Dates(the_dates)
  ) %>%
    mutate_at(-1, as.Date) 
})
```

``` r
knitr::kable(res, caption = "Les resultats", label = 3)
```

| original         | base       | lubridate  | readr      | anytime    | parsedate  | date       | datetimeutils | rccdates   |
|:-----------------|:-----------|:-----------|:-----------|:-----------|:-----------|:-----------|:--------------|:-----------|
| 2020-02-04       | 2020-02-04 | 2020-02-04 | 2020-02-04 | 2020-02-04 | 2020-02-04 | NA         | NA            | 2020-02-04 |
| 04 February 2020 | NA         | 2020-04-20 | 2020-02-04 | 2020-02-04 | 2020-02-04 | 2020-02-04 | NA            | NA         |
| 2/4/20           | 2020-02-04 | 2020-02-04 | 2020-02-04 | NA         | 2020-02-04 | 1920-02-04 | NA            | NA         |
| 4/2/20           | 2020-02-04 | 2020-04-02 | 2020-02-04 | NA         | 2020-04-02 | 1920-04-02 | NA            | NA         |
| 04 Février 2020  | 2020-02-04 | 2020-04-20 | 2020-02-04 | NA         | 2020-01-04 | NA         | NA            | NA         |
| 43865            | NA         | NA         | NA         | 4386-01-01 | 2021-12-27 | NA         | NA            | NA         |

Table 3: Les resultats

# No right answer

I think the main thing to take away from this exercise is that there is no right
answer when it comes to parsing dates. I’ve helped out with [a project that aims
at trying to provide yet another magic
solution](https://www.repidemicsconsortium.org/linelist/reference/guess_dates.html),
but even it has drawbacks. **You ultimately get the best results when you know
the formats and locales you are dealing with,** so the best option is just to
evangelize about the ISO 8601 and hope. All the available packages have their
own idiosyncrasies and it’s an absolute minefield when it comes to dates (note
that this doesn’t even get into things like [the year 2038
problem](https://en.wikipedia.org/wiki/Year_2038_problem), or [leap
seconds](https://en.wikipedia.org/wiki/Leap_second)).

[^1]: Note that these [may be mixed in with text-based dates in Excel, making them even more difficult to parse](https://github.com/everhartlab/SscPhenoProj/blob/54c8d75ddb917866e94c06f359f3bc2b61b8de7f/Analysis.R#L192-L207)

[^2]: This is a bit of a stretch since lubridate provides tools for manipulating timespans
