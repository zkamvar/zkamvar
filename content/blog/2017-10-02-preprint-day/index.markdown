---
title: Preprint Day!
author: Zhian N. Kamvar
date: '2017-10-02'
slug: preprint-day
categories:
  - science
tags:
  - preprint
  - reproducibility
  - docker
  - GitHub
  - Open Science Framework
  - Continuous Integration
banner: img/banners/preprint-screencap.png
---

Today, we finally published a preprint for [a manuscript that spans over 15 years of data](https://peerj.com/preprints/3311/?td=bl) 🎉!

{{% tweet "915033532543770624" %}}

Here’s the citation:

> Kamvar ZN, Amaradasa BS, Jhala R, McCoy S, Steadman JR, Everhart SE. (2017) Population structure and phenotypic variation of *Sclerotinia sclerotiorum* from dry bean in the United States. *PeerJ Preprints* 5:e3311v1 <https://doi.org/10.7287/peerj.preprints.3311v1>

------------------------------------------------------------------------

This project started in 2003 as a way to screen new dry bean lines for resistance to [white mold](http://extension.colostate.edu/topic-areas/agriculture/white-mold-of-dry-beans-2-918/) by growing these lines in screening nurseries[^1][^2] with unsuppressed populations of *S. sclerotiorum* (the causal agent of white mold).
Over the years, [sclerotia](http://www.sclerotia.org/lifecycle/sclerotial-development) were collected from these fields and stored in the collection at UNL.

We used the collection from these white mold screening nurseries along with isolates from grower fields to test if the populations of *S. sclerotiorum* in the screening nurseries were genetically differentiated from populations in the surrounding regions.

## Data Analysis

For this paper, I decided to go full reproducibility.
This means that I have all of the scripts, data, and code available (at <https://github.com/everhartlab/sclerotinia-366#readme>).
All of the figures, tables, and numbers in the manuscript are generated directly from the analyses.
There was almost no copying and pasting on my part.
It was a lot of effort, but seriously the figures turned out amazing!

<img src="graph-11-loci-1.png" width="161" style="display: block; margin: auto;" />

What’s more is that because the code is available, all the instructions for recreating this figure are available: <https://github.com/everhartlab/sclerotinia-366/blame/master/doc/RMD/MLG-distribution.Rmd>

All the results and manuscript is controlled via [Makefile](http://kbroman.org/minimal_make/).
This means that you can see [exactly how the scripts are to be run](https://github.com/everhartlab/sclerotinia-366/blob/master/Makefile).
I included a [DESCRIPTION](https://github.com/everhartlab/sclerotinia-366/blob/master/DESCRIPTION) file in the repository so you can automatically install the necessary packages by using [devtools](https://github.com/hadley/devtools#readme):

``` r
# install.packages("devtools")
devtools::install_github("everharthlab/sclerotinia-366", repos = "https://mran.microsoft.com/snapshot/2017-09-30")
```

Because we linked our project with the [Open Science Framework](https://osf.io), you can also cite the code and data:

> Kamvar, Z. N., Amaradasa, B. S., Jhala, R., McCoy, S., Steadman, J., & Everhart, S. E. (2017, October 3). Population structure and phenotypic variation of *Sclerotinia sclerotiorum* from dry bean in the United States. <http://doi.org/10.17605/OSF.IO/EJB5Y>

### Docker

Perhaps the coolest thing about this project was my first foray into using [Docker](http://seankross.com/2017/09/17/Enough-Docker-to-be-Dangerous.html) as a way to improve reproducibility.
I created a [Dockerfile](https://github.com/everhartlab/sclerotinia-366/blob/master/Dockerfile) that gets rebuilt with [Circle CI](https://circleci.com/gh/everhartlab/sclerotinia-366) and updated on [Docker Hub](https://hub.docker.com/r/zkamvar/sclerotinia-366/) every time I make a change to the repository.
This is really cool because you can download and explore the analysis without having to install anything more than just Docker.

To explore the full analysis, you can download docker and type in your terminal:

``` bash
docker run --rm -dp 8787:8787 zkamvar/sclerotinia-366:latest
# 1f3e92ec0378c0a44cce63990a4b07f78f38bc0d9ecfd3b19ea042d33e892be4
```

Once the hash shows up, you can go to your browser and type `localhost:8787` and an Rstudio window will open (if you’re presented with a login screen, login with user: rstudio, pass: rstudio):

<figure>
<img src="../../img/docker-rstudio1.png" style="width:100.0%" alt="Rstudio startup" /><figcaption aria-hidden="true">Rstudio startup</figcaption>
</figure>

The analysis is in the `/analysis` directory, but since that’s not writable, you should copy it to the home directory.
In Rstudio type:

``` r
system("cp -R /analysis .")
```

And you should see an analysis folder pop up in your Files pane.
Double click on that, scroll all the way down and double click on `znk_analysis.Rproj`, Rstudio will ask you if you want to switch to that project.

Now, you can navigate to the RMD documents in `doc/RMD` or the manuscript in `doc/manuscript/manuscript.Rmd` and play around.

If you type <kbd>CTRL + .</kbd>, you can navigate to `mlg-mcg.Rmd`, where you can
re-execute all of the code by selecting “Run All” from the Run drop-down:

<figure>
<img src="../../img/docker-rstudio2.png" style="width:100.0%" alt="Run all" /><figcaption aria-hidden="true">Run all</figcaption>
</figure>

Once everything runs, you have all the data from that Rmarkdown document to inspect and manipulate.

<img src="../../img/docker-rstudio3.png" style="width:100.0%" />

You can even copy the analysis to your computer while the image is running:

``` bash
docker ps
# CONTAINER ID        IMAGE                            COMMAND             CREATED             STATUS              PORTS                    NAMES
# 1f3e92ec0378        zkamvar/sclerotinia-366:latest   "/init"             24 minutes ago      Up 24 minutes       0.0.0.0:8787->8787/tcp   modest_cori
docker cp modest_cori:/analysis .
```

Once you’re finished, you can close that browser window/tab and then kill your container using `docker stop`:

``` bash
docker ps
# CONTAINER ID        IMAGE                            COMMAND             CREATED             STATUS              PORTS                    NAMES
# 1f3e92ec0378        zkamvar/sclerotinia-366:latest   "/init"             24 minutes ago      Up 24 minutes       0.0.0.0:8787->8787/tcp   modest_cori
docker stop modest_cori
# modest_cori
```

Seriously. ✨ This is really awesome ✨

So, I could have gone with a different method for reproducibility such as [packrat](https://rstudio.github.io/packrat/), but I have [previously been burned by it](https://github.com/zkamvar/Sudden_Oak_Death_in_Oregon_Forests#readme).
Specifically, it would do this thing where, when you opened up the repo, it would boot itself and attempt to load or build the packages for the project without asking.
This was a pain in the butt back in 2014 when [rgdal](https://github.com/zkamvar/Sudden_Oak_Death_in_Oregon_Forests#exceptions) support was flimsy on OSX.
For that reason, I chose to go the route of the DESCRIPTION file and Docker route.

[^1]: https://naldc.nal.usda.gov/naldc/catalog.xhtml?id=IND43757287

[^2]: https://doi.org/10.1094/PDIS-11-10-0865
