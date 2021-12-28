---
title: Using a custom library in R
author: Zhian N. Kamvar
date: '2017-11-18'
slug: using-a-custom-library-in-r
categories:
  - R
  - example
tags:
  - library
  - .libPaths()
  - .Renviron
  - environment variable
  - R
banner: img/banners/smashing-library.gif
---



I've been using a custom library for R since 2012 and I've never looked back. 
I've not seen many tutorials for people do do this through R, so I figured I'd
write a quick one.

# Where does your R package library live?

You can usually find this out by typing `.libPaths()` in your R console. If you have an out-of-the-box installation, it will generally be somewhere like:

```
C:/Program Files/R/R-3.4.2/library
```

or

```
/Library/Frameworks/R.framework/Versions/3.4/Resources/library
```

This is the place on your computer where R will put any packages you attempt
to install with `install.packages()`. However, you might notice one problem:
this library is version-specific, which means that when you update R, you will
also have to [migrate your existing
library](https://www.r-bloggers.com/upgrade-r-without-losing-your-packages/).
Moreover, if you're on Windows, [you may not have permissions to write to the
library path](https://stackoverflow.com/q/15170399/2752888) 🙀.

# What if I told you there was a better way?

![](img/banners/smashing-library.gif)

The solution to avoid the hassle of constant library migration is to make a custom library folder[^0] and set the environmental variable 
`R_LIBS` to point to that folder 👍. 

Now, this has been
[touched](https://www.r-bloggers.com/updating-r-but-keeping-your-installed-packages/)
[on](https://csgillespie.github.io/efficientR/3-3-r-startup.html#renviron)
[before](http://kbroman.org/pkg_primer/pages/build.html#installing-a-package-in-a-personal-directory),
but when it comes to explaining HOW to set that environment
variable, something is always missing[^1] 🤷‍♂. This short post will demonstrate how to persistently set the
variable *without ever having to leave R* 😎.

# Creating your `.Renviron` file and setting `R_LIBS`

The `.Renviron` file is where you can store environment variables that R uses
to gather specific information about your particular computer. In this case, we
are going to use it to tell R where your R library lives using the `R_LIBS`
environment variable. 

What we are going to do is:

1. Create a folder (~/R/library) to serve as our new library
2. Create a file called `~/.Renviron`[^2]
3. Add `R_LIBS=~/R/library` to the `~/.Renviron` file
4. Restart R and install our packages.

> For this tutorial, I'm using ~/R/library for the custom library, but you can
> set it to any folder you wish.

The first step, create the directory. Note that this will give you a warning
message if the directory already exists.


```r
dir.create("~/R/library", recursive = TRUE)
```

Now we can create our `~/.Renviron` file. This is a part we need to be careful.
If there already is a `.Renviron` file, we don't want to accidentally clobber
it. To help with that, we will use `file.exists()` with an `if` statement to 
tell us whether or not to create the file:


```r
if (file.exists("~/.Renviron")) {
  message("~/.Renviron already exists!")
} else {
  file.create("~/.Renviron")
  message("created ~/.Renviron")
}
```

```
## ~/.Renviron already exists!
```

Now we can write `R_LIBS=~/R/library` to the `~/.Renviron` file:


```r
cat("R_LIBS=~/R/library", file = "~/.Renviron", append = TRUE)
```

## Restart R

Now after you've restarted R, your library will appear:


```r
.libPaths()
```

```
## [1] "/Users/zhian/R"                                                
## [2] "/Library/Frameworks/R.framework/Versions/4.1/Resources/library"
```


# Moving forward

Now you can feel free to update your version of R without having to go through
the hassle of migrating your library. Even better, you can take advantage of
`update.packages()` to auto-update packages in your R library like so:


```r
update.packages(ask = FALSE, checkBuilt = TRUE)
```


[^0]: Note: I am using the word "folder" here to mean "directory".
[^1]: Please do not take this to mean that these are bad resources. I'm just picking nits here 😄. 
[^2]: Two things: 1) The ~/ denotes the home folder. All of your documents are nested in this folder. 2) `.Renviron` starts with a dot, which hides it from your file browser by default.
