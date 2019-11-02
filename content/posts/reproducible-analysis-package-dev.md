---
title: 'Reproducible analysis: package development'
date: 2019-11-02T11:12:48+01:00
draft: false
---

## Issue

[Packages](http://r-pkgs.had.co.nz/intro.html) are used to share your R code with others. Why then should this be the first topic tackled? Couldn't we make our code shareable after we made something to actually share? Well, I actually don't make the analysis a package mainly for sharing it (even though it's nice to have). Packages also have the very nice side effect they enforce conventions. Other packages (e.g. `testthat`) build on these conventions. If we adhere to these conventions we'll have way less issues interfacing with other people's code. Conventions of course have an impact on nearly everything so it makes sense to make our analysis a package from the start.

## Solution

In the R ecosystem `devtools` is the de facto standard to build packages. `devtools` helps with common tasks like scaffolding, running tests, ... I base myself mainly on the explanation in the [R packages book](http://r-pkgs.had.co.nz/intro.html). References to this book will popup later on as well when talking about tests or documentation.

At the moment there are 2 slightly different versions of the R packages book available. The [first version](http://r-pkgs.had.co.nz/) is a bit outdated (published in April 2015) focusing on an older version of `devtools`. The [second version](https://r-pkgs.org/index.html) is based on `devtools 2.0.0`. It's more up to date but less complete at the moment. I try as much to work with the second edition as possible. If some part is not yet available in the second edition I base myself on the first edition.

## Creating package

To use `devtools` to create a package we first have to load it in the console:

```
library(devtools)
```

Let's go to the root of our `reproducible-analysis` repository and run the following in the console to create a package:

```
create_package("reproseries")
```

This doesn't work:

![devtools create package in existing folder](/devtools-create-existing-folder-prompt.png)

At first I tried to come up with ways to turn existing code into an R package. I was sorely disappointed by the lack of documentation on that subject. However, this is probably a good thing since it forces you to start from a clean slate which is what I did. Now the `reproducible-analysis` repository is abandoned in favor of `reproseries`.

Instead of running in the root of `reproducible-analysis` run this where you want to create the package:

```
create_package("reproseries")
```

Since there was no directory `reproseries` yet this works fine. Some files are created automatically of which the most important are:

- R
- DESCRIPTION
- .gitignore

The `R` folder contains the functions used in the analysis, `DESCRIPTION` gives metadata about the package: its title, a short description of what it does, its version, ...

Note the `.gitignore` file being there doesn't mean the package is a Git repository yet.

## Use Git

Run `use_git()` in the console. Git and version control will be covered in a later post but what it does is:

- initialize repo
- add all files to staging
- commit them as first commit

I could have done the same on the command line:

- `git init`
- `git add -A`
- `git commit -m "Initial commit"`

However, why not use shortcuts if they're available and clear?

## Migrate functions to R

[Last post](https://isaacverm.github.io/posts/reproducible-analysis-example/) we already wrote some functions for our analysis. It's time to migrate these from the `reproducible-analysis` repository to the `reproseries` package.

Now is a good moment to think about how to structure these functions. Maybe we want to group them by the data analysis step they belong to (`import-functions.R`, `tidy-functions.R`, ...)? Or maybe group them by the data they manipulate (`temperatures.R`, `avg_temperatures.R`)? It's not a clear cut choice but I noticed from experience consistency is key. Think about the grouping at the beginning of the project and stick with it not to confuse yourself. It's very tempting to half way start rearranging everything. Only does it if it's really necesary and you know where you want to end up. In this case I think it makes a lot of sense to group by data analysis step.

The philosophy of `devtools` is not to "hit the metal" yourself but let functions do the heavy lifting for you. So instead of copying the code ourselves to the `R` folder let's get some help from the `use_r` function. Running `use_r("tidy")` opens the `tidy.R` file in the `R` folder:

![tidy function in r folder](/tidy-function-in-r-folder.png)

You might notice the `tidyNottem` function uses two other functions, `tk_tbl` from the `timetk` package and `separate` from the `tidyr` package (part of `tidyverse`). In the original `analysis.R` script we defined these dependencies with `library`. However, you [don't do it that way in a package](http://r-pkgs.had.co.nz/r.html). Packages are loaded in two distinct steps:

- build
- the actual loading

If you add a `library` call in your functions file this will only make the package available in the build phase but not in the actual load phase. To have the package available in the load phase you have to add it to the `DESCRIPTION` [file](http://r-pkgs.had.co.nz/description.html#dependencies) with the `Imports` field:

![imports description example](/imports-description-example.png)

Note `magrittr` has to be imported to have access to the `%>%` pipe operator. [Modify](https://stackoverflow.com/questions/27947344/r-use-magrittr-pipe-operator-in-self-written-package) the `NAMESPACE` adding . I actually have no idea why it works with the [namespace](http://r-pkgs.had.co.nz/namespace.html) but didn't by just adding it to `imports` in the `DESCRIPTION`. Oh well.

![magrittr import exception](/namespace-magrittr.png)

Other fields like license I leave untouched at the moment. It's created by default. You can see keeping tracks of external packages used can quickly become a hassle:

At the moment I still remember `separate` belongs to the `tidyverse` and `tk_tbl` is part of `timetk`. I'm quite sure I won't remember this in a few days, I dread talking about what I'll remember about it in a few months. So it's best practice to add the package name to the function name like this:

![function calls with double semicolon example](/function-calls-with-double-semicolon-example.png)

This does get verbose. An alternative solution would be to use [namespace imports](http://r-pkgs.had.co.nz/namespace.html#imports) but that would lead us too far. In general I don't mind it makes the function calls long Try to look on the bright side. Since you lose a number of characters by default at the beginning you have to think harder about concise and practical function names.

Now let's try out if our tidy function does something by running this in the console:

```
library(devtools)
load_all()
tidyNottem(nottem)
```

`load_all` is a `devtools` function making all functions in the package available for interactive use. `nottem` is a built-in dataset so need to import it explicitly.

The process of migrating the other functions follows the same patterns as for the tidy function:

- `use_r()` to create file
- copy function definition to file (but no `library` calls)
- `load_all` to test if function is available

## Examples

## Summary

Run `create_package()` to create a package. Do this from scratch and not an existing directory.

Run `use_git()` to quickly get a repo set up.

The problems with creating a package from an existing repo clearly show:

- it pays off to think about what issues might arise IN ADVANCE
- even with a small code base things can get funky real fast if you don't get a grip

## Sources

https://hilaryparker.com/2014/04/29/writing-an-r-package-from-scratch/
