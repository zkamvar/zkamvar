---
title: Parsing GitHub Task Lists with {tinkr}
author: Zhian N. Kamvar
date: '2023-04-13'
slug: gh-task-lists
categories:
  - github
  - R
  - example
tags:
  - tinkr
  - XML
  - XPath
  - GitHub
  - misc
  - R
subtitle: ''
excerpt: ''
draft: false
series: ~
layout: single
---

Oh hey! It's been a while. I have but a moment, but I wanted to share something
cool. The GitHub API does not parse task lists in issues, but I found a way to
leverage the {tinkr} package (authored by Maëlle Salmon and myself) to parse out
the tasklists with the power of XPath!

I have a big project where I need to track [a bunch of task lists in GitHub 
issues](https://github.com/carpentries/lesson-transition/labels/lesson). The
[GitHub REST API for issues](https://docs.github.com/en/rest/issues?apiVersion=2022-11-28) 
allows me to get the the issues I'm interested in by tag.

Here, I'm filtering the issue by the tags "lesson" and "early transition", which
are issues I need to address like right now

``` r
library("gh")
issues <- gh("GET /repos/carpentries/lesson-transition/issues", .params = list(labels = "lesson,early transition"), per_page = 100, .limit = Inf)
length(issues)
#> [1] 12
names(issues[[1]])
#>  [1] "url"                      "repository_url"          
#>  [3] "labels_url"               "comments_url"            
#>  [5] "events_url"               "html_url"                
#>  [7] "id"                       "node_id"                 
#>  [9] "number"                   "title"                   
#> [11] "user"                     "labels"                  
#> [13] "state"                    "locked"                  
#> [15] "assignee"                 "assignees"               
#> [17] "milestone"                "comments"                
#> [19] "created_at"               "updated_at"              
#> [21] "closed_at"                "author_association"      
#> [23] "active_lock_reason"       "body"                    
#> [25] "reactions"                "timeline_url"            
#> [27] "performed_via_github_app" "state_reason"
```

There are 12 issues that need my attention right now and each one will have a
body element that contains the body of the top-level issue. Take for example
https://github.com/carpentries/lesson-transition/issues/79:


```r
issues[[1]][c("title", "number", "body")]
#> $title
#> [1] "swcarpentry/r-novice-gapminder-es"
#> 
#> $number
#> [1] 79
#> 
#> $body
#> [1] "tracking issue for <https://github.com/swcarpentry/r-novice-gapminder-es>\r\n\r\n - [x] build error due to [stray `.r` class item](https://github.com/swcarpentry/r-novice-gapminder-es/blob/fb174938184b4425fa378b3b388e77a18a52dc7c/_episodes_rmd/07-control-flow.Rmd#L330) (to be fixed in https://github.com/swcarpentry/r-novice-gapminder-es/pull/136)\r\n - [ ] build error due to stray index and references (to be fixed in https://github.com/swcarpentry/r-novice-gapminder-es/pull/141)\r\n   ```r\r\n   Error in vapply(episodes, get_titles, character(1)) : \r\n     values must be length 1,\r\n    but FUN(X[[17]]) result is length 0\r\n   ```\r\n - [x] fix glossary definition list\r\n - [ ] fix missing file in reference.md titles (`04-Estructura-de-datos-parte1/` and `05-Estructura-de-datos-parte2/`)\r\n - [ ] fix renv errors \r\n \r\n<details><summary>renv error details</summary>
 ... SNIP ...
</details> \r\n\r\n"
```

We can print the body to markdown and show what it looks like:

``` r
library(gh)
issues <- gh("GET /repos/carpentries/lesson-transition/issues", .params = list(labels = "lesson,early transition"), per_page = 100, .limit = Inf)

issues[[1]]$body |> writeLines()
#> tracking issue for <https://github.com/swcarpentry/r-novice-gapminder-es>
#> 
#>  - [x] build error due to [stray `.r` class item](https://github.com/swcarpentry/r-novice-gapminder-es/blob/fb174938184b4425fa378b3b388e77a18a52dc7c/_episodes_rmd/07-control-flow.Rmd#L330) (to be fixed in https://github.com/swcarpentry/r-novice-gapminder-es/pull/136)
#>  - [ ] build error due to stray index and references (to be fixed in https://github.com/swcarpentry/r-novice-gapminder-es/pull/141)
#>    ```r
#>    Error in vapply(episodes, get_titles, character(1)) : 
#>      values must be length 1,
#>     but FUN(X[[17]]) result is length 0
#>    ```
#>  - [x] fix glossary definition list
#>  - [ ] fix missing file in reference.md titles (`04-Estructura-de-datos-parte1/` and `05-Estructura-de-datos-parte2/`)
#>  - [ ] fix renv errors 
#>  
#> <details><summary>renv error details</summary>
#  ... SNIP ...
#> </details> 
#> 
```


I can see here that I have a tasklist, but the GitHub API does not provide it
for me. Luckily, I have the {tinkr} package, which can read in Markdown and
parse it to XML. All I have to do is write the body to a temporary file and then
read it in:

``` r
tmp <- tempfile()
issues[[1]]$body |> writeLines(tmp)
the_task <- tinkr::yarn$new(tmp)
xml2::xml_find_all(the_task$body, ".//md:tasklist", ns = the_task$ns)
#> {xml_nodeset (5)}
#> [1] <tasklist completed="true">\n  <paragraph>\n    <text xml:space="preserve ...
#> [2] <tasklist completed="false">\n  <paragraph>\n    <text xml:space="preserv ...
#> [3] <tasklist completed="true">\n  <paragraph>\n    <text xml:space="preserve ...
#> [4] <tasklist completed="false">\n  <paragraph>\n    <text xml:space="preserv ...
#> [5] <tasklist completed="false">\n  <paragraph>\n    <text xml:space="preserv ...
```


Knowing this, I can write a function that will take an issue response from GitHub
and extract the tasklist into a dataframe. I'm not going through the entire genesis,
but here's the outcome:

``` r
#' Extract tasklist from issue
#' 
#' This will take a GitHub issue and extract any tasklist into a data frame
#' 
#' @param issue a list representing a single issue from the GitHub issues API
#' @return a data frame with the following columns:
#'   - **lesson**   the name of the issue (in this case it corresponds to a lesson)
#'   - **issue**    the number of the issue
#'   - **task**     a specific task in the issue
#'   - **complete** a logical indicator if the task was completed
#'   - **url**      url to the issue
extract_tasklist <- function(issue) {
  title <- issue$title
  nr <- issue$number
  url <- issue$html_url
  # NOTE: use a text connection to save on I/O
  f <- textConnection(issue$body)
  on.exit(close(f), add = TRUE)
  y <- tinkr::yarn$new(f)
  # allow us to parse issues without any tasks
  status <- as.logical(NA)
  complete <- 0
  total    <- 0
  tasks <- xml2::xml_find_all(y$body, ".//md:tasklist", ns = y$ns)
  if (length(tasks)) {
    status <- xml2::xml_attr(tasks, "completed") == "true"
    complete <- sum(status)
    total <- length(status)
    tasks <- xml2::xml_text(tasks)
  } else {
    tasks <- NA_character_
  }
  msg <- "{complete}/{total} tasks complete: (#{sprintf('%02d', nr)}) {title}"
  if (complete == total) {
    complete <- cli::style_bold(cli::col_blue(complete))
    cli::cli_alert_success(msg)
  } else {
    cli::cli_alert_info(msg)
  }

  tibble::tibble(lesson = title, issue = nr, task = tasks, 
    complete = status, url = url)

}
```


And now I can create a data frame of tasks for my issue, tallying the number of
completed tasks:

```r
extract_tasklist(issues[[1]])
#> ℹ 2/5 tasks complete: (#79) swcarpentry/r-novice-gapminder-es
#> # A tibble: 5 × 5
#>   lesson                            issue task                    complete url  
#>   <chr>                             <int> <chr>                   <lgl>    <chr>
#> 1 swcarpentry/r-novice-gapminder-es    79 "build error due to st… TRUE     http…
#> 2 swcarpentry/r-novice-gapminder-es    79 "build error due to st… FALSE    http…
#> 3 swcarpentry/r-novice-gapminder-es    79 "fix glossary definiti… TRUE     http…
#> 4 swcarpentry/r-novice-gapminder-es    79 "fix missing file in r… FALSE    http…
#> 5 swcarpentry/r-novice-gapminder-es    79 "fix renv errors"       FALSE    http…
```

From here, I can also run `purrr::map_dfr()` over all the issues in order to 
create a table that will help me search for specific patterns that can help me
determine if it's an issue that I need to fix globally for several lessons or if
it needs to be fixed locally just in this lesson.