---
title: What methods do I need in an S3 object?
author: Zhian N. Kamvar
date: '2019-07-19'
slug: what-do-i-need-in-an-s3-object
categories:
  - R
  - good-practice
tags:
  - S3
  - programming
  - R
  - functions
  - packages
  - oop
draft: true
---

For the past few months, I've been working on the [R4EPIs] project, in which we
are trying to create standardized templates and training for field
epidemiologists (epis) to use R in their work. One of the challenges we ran
into during this project was the need to aggregate weekly data starting on
Saturdays. The only packages (that we knew of) to handle weeks were either
[ISOweek] and [lubridate], and they could only handle weeks starting on Monday
and Sunday[^1]. After not being able to find a simple way to perform this task,
I decided to create the [aweek] package to [easily translate dates to weeks]
and back again. 

In order to complete this task, I knew that I needed to create a new R object
that could hold ISO-formatted weeks (e.g. 2019-W29-5),
and remember which day the week started on. I settled on the concept of adding
an attribute called `week_start` to a character vector. This attribute would be
an integer from 1 to 7, representing the day of the ISO week that this
representation of a week started on. This way, R would always know how to
convert that week to a date. 

I had followed the S3 section of the [OO field guide], for creating an object
and defined a few methods for it: `print.aweek`, `as.Date.aweek`, 
`as.POSIXt.aweek` and `as.character.aweek`. I wrote code, wrote tests, and 
validated the conversions from dates to weeks and back again and was largely 
satisfied with the result 😁... that is until I tried to [subset 
the objects] 🙈:

```r
library("aweek")
d <- Sys.Date()
w <- date2week(d)
str(w)
#>  'aweek' chr "2019-W10-4"
#>  - attr(*, "week_start")= int 1
str(w[1])
#>  chr "2019-W10-4"
```

It turns out that you need to define the `[` and `c` methods for your S3
objects if you want to be able to slice and combine them, respectively. I fixed
that bug, but began to run into other bugs that dropped the aweek class such as
subsetting with `[[` or attempting to [convert to data frames]. I realized
that I wasn't sure exactly *what* methods I should use to make sure my class was
robust to things people would possibly want to do with it and decided to figure
out what other classes did, and I found somewhere that the `methods()` function
exists and will show you all the public methods for an S3 object:


```r
methods(class = "Date")
```

```
##  [1] -             !=            [             [[            [<-          
##  [6] +             <             <=            ==            >            
## [11] >=            as.character  as.data.frame as.list       as.POSIXct   
## [16] as.POSIXlt    Axis          c             coerce        cut          
## [21] diff          format        hist          initialize    is.numeric   
## [26] julian        length<-      Math          mean          months       
## [31] Ops           pretty        print         quarters      rep          
## [36] round         seq           show          slotsFromS3   split        
## [41] str           summary       Summary       trunc         update       
## [46] weekdays      weighted.mean xtfrm        
## see '?methods' for accessing help and source code
```

That gave me a place to start because I realized immediately that I didn't have
a `[<-` or `[[` method defined, which was pretty important. I ultimately settled
on the following [methods for aweek objects]:


```r
library("aweek")
methods(class = "aweek")
```

```
##  [1] [             [[            [<-           as.aweek      as.character 
##  [6] as.data.frame as.Date       as.list       as.POSIXlt    c            
## [11] print         rep           trunc        
## see '?methods' for accessing help and source code
```

Out of these, the `[`, `[[`, `[<-`, and `c` methods are probably the most
important because that's ultimately how users will be interacting with these
objects, everything else comes down to what specific features your object has. 




[R4EPIs]: https://blogs.msf.org/bloggers/larissa/innovation-introducing-r4epis
[aweek]: https://www.repidemicsconsortium.org/aweek
[easily translate dates to weeks]: https://www.repidemicsconsortium.org/2019-06-12-aweek-1.0.0/
[ISOweek]: https://cran.r-project.org/package=ISOweek
[lubridate]: https://cran.r-project.org/package=lubridate
[OO field guide]: http://adv-r.had.co.nz/OO-essentials.html#s3
[subset the objects]: https://github.com/reconhub/aweek/issues/1
[convert to data frames]: https://github.com/reconhub/aweek/issues/8
[methods for aweek objects]: https://github.com/reconhub/aweek/blob/bb5bfbdcf30bfec3318cfa4cc1023fba961df63d/NAMESPACE#L3-L21
[^1]:  we found out later that lubridate COULD actually convert dates to week numbers starting on any day, but it wasn't precisely what we needed]. 
