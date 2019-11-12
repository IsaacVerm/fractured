---
title: "Reproducible analysis: literate programming"
date: 2019-11-12T18:43:48+01:00
draft: false
---

## Issue

When starting off as a beginner doing data analysis you often start with hacking together some scripts. You pick whatever you can find, throw it in the mix and let everything run. These scripts create output like plots, tables, ... which you manually copy afterwards into some document. There are several reasons why this is frankly an excruciating way of working:

- if some output changes you have to manually copy the changed results to the document where you write the analysis
- changing some parameters of your analysis requires rerunning everything
- the explanation of what your code does is totally decoupled from the code itself

All this doesn't really favor reproducible analysis. Hence a paradigm shift, [literate programming](http://www.literateprogramming.com/). Literate programming means you explain what your code does at the same time and place where you write the code. An excellent way to put this into practice is [using a package structure to write your analysis](https://rmflight.github.io/posts/2014/07/analyses_as_packages.html) with [vignettes](http://r-pkgs.had.co.nz/vignettes.html). Vignettes combine code and descriptive text in the same document using [rmarkdown](https://rmarkdown.rstudio.com/lesson-1.html). Vignettes can also be output to html which makes it ideal to be published as blog posts. Even more, it's an excellent way of doing it since you can tap into all a package has to offer (testing, documentation) which greatly benefits your analysis as well.

## Solution

Vignettes are supported by default in `devtools`.

## Create the vignette

First of all we use `devtools` to create a vignette. This vignette will be our analysis file.

```
use_vignette("nottingham-temperatures")
```

Note you can't run `devtools:use_vignette` anymore as described in the [R packages](http://r-pkgs.had.co.nz/vignettes.html) book. `devtools` is no longer the owner of this function, it now belongs to `usethis`. [usethis](https://usethis.r-lib.org/reference/index.html) offers plenty more interesting functionality, integration with GitHub (as shown in the [package development](https://isaacverm.github.io/posts/reproducible-analysis-package-dev/) post) and Travis among other things.

As expected from `devtools` it does everything necessary to start with a vignette:

- intermediary build files are automatically added to `.gitignore`
- dependencies are added to `DESCRIPTION`
- and most importantly the `nottingham-temperatures.Rmd` file has been created

The `.Rmd` file generated contains some standard setup for knitr (the vignette engine we use). A [vignette engine](https://www.rforge.net/doc/packages/knitr/vignette_engines.html) transforms the input file (in our case in `rmarkdown` format) to the right output (`html` by default and that's also what we're aiming for).

## Writing the analysis

We'll use the vignette as a way to show how the functions we wrote previously can be combined and what their output is. The vignette itself is divided in multiple parts:

- a small introduction
- an explanation of what the input data looks like
- going through the functions in each analysis phase

Some defaults for the image size seem to be off but can be [modified](https://sebastiansauer.github.io/figure_sizing_knitr/).

There actually aren't a lot of really important things to say about writing vignettes. It's just a combination of R and Markdown. This [cheatsheet](https://rstudio.com/wp-content/uploads/2015/02/rmarkdown-cheatsheet.pdf) shows most of the options available to you. It doesn't make a lot of sense to go over all of them. Best thing is just to look them up when you need them. Options exist to hide your code, cache results, show warnings or not, ...

This is the complete analysis in [rmarkdown format](https://github.com/IsaacVerm/reproseries/blob/793068abf1ee7920e1f97bdecbbab5c2ceb90b86/vignettes/nottingham-temperatures.Rmd).

## Generating the vignette

Run `devtools::document`. This also updates the export tags of your functions.

Important note to myself: you should [export your functions](https://stackoverflow.com/questions/35727645/devtools-build-vignette-cant-find-functions)!

So:

- add an `@export` tag to each function
- run `devtools::document()` to update the `NAMESPACE`

If not your functions will only be available for interactive use after `devtools::load_all()` but not in the vignette.
