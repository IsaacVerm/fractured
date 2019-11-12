---
title: 'Reproducible analysis: documentation'
date: 2019-11-12T14:45:00+01:00
draft: false
---

## Issue

We wrote the code, we wrote the tests but we didn't specify explicitly how the code can be used. Take for example the `calculateAutoCorrelation` function. From its name we can infer it should return some kind of auto correlation coefficient. But how exactly is this correlation calculated? What kind of lag we take into account? Those questions can more easily be answered if we add some [metadata](http://r-pkgs.had.co.nz/man.html) to the functions we created.

In this post I won't go into documentation in general but will focus specifically on how to document functions in R. So READMEs, vignettes, won't be covered here (although vignettes will come up when talking about literate programming).

## Solution and alternatives

Yet again we use what's suggested by `devtools` so we'll stick to `roxygen2`. This package helps with combining code and documentation in the same place. I didn't explore other alternatives.

## Adding a comment

As an example I'll show how the `plotAvgTemperaturesByYear` in `visualize.R` is documented.

Comments consist of two parts:

- an introduction (specified implicitly)
- tags (specified explicitly)

### Introduction

Everything before the first tag is considered the introduction. The introduction consists of:

- a title
- a description
- details

The title and description are required so you can't document a title without also documenting a description.

This is how it's put into practice:

```
#' Plot of average temperatures by year
#'
#' \code{plotAvgTemperaturesByYear} plots the average temperature by year in a line graph.
plotAvgTemperaturesByYear <- function(avg_temperatures,
                                      place = "Nottingham") {
                                          ...
```

As you can see I don't use any explicit tags saying this is a title, this is a description, ... By default the first line is the title, the second line is the description and any following lines are details. Lines are separated with `#'`.

You have to specify the function name with `\code{}` but this seems kind of superfluous since you're already documenting a function so by definition you know its name.

Without adding any specific tags we can already create the documentation by typing the following in the console:

```
devtools::document()
```

This creates a `.Rd` file in a `man` folder. You don't actually need to know how this file works since you always manipulate it by documenting functions right next to those functions.

Now if you type `?plotAvgTemperaturesByYear` in the console you get a small summary of what the function does. This works exactly the same as it does for built-in functions like `sum`, `mean`, ...

![basic roxygen doc example](/basic-roxygen-doc-example.png)

We didn't have to specify it explicitly but `roxygen2` adds some information about usage and what package the function belongs to.

### Tags

The function is still poorly documented of course. In this post I won't go over all the existing tags cause these are already [well documented](http://r-pkgs.had.co.nz/man.html). When documenting functions you can get by by using just 3 tags:

- `@param` (to document input)
- `@return` (to document output)
- `@examples` (self-explanatory)

Let's make things a bit more precise:

```
...
#' @param avg_temperatures Data.frame. Has years and their corresponding average temperature.
#' @param place String. It's assumed this function is called with the nottem built-in dataset.
#' This dataset records the temperatures in Nottingham. Nottingham is the default argument.
#' You can override this argument if the need arises.
#' @return ggplot object
#' @examples
#' avg_temperatures <- data.frame(year = c(1920, 1921, 1922, 1923),
#'                                avgTemperature = c(5, 10, 8, 9))
#' plotAvgTemperaturesByYear(avg_temperatures)
#' @export
plotAvgTemperaturesByYear <- ...
```

Important note: the `@export` is essential. If not the function is not added to the [namespace](https://stackoverflow.com/questions/44722297/could-not-find-function-in-roxygen-examples-during-cmd-check) and checking using `devtools::check()` will fail. This is an issue because examples are more than just text, they're actually executed each time you run `devtools::check()` to make sure they're at least executable (it's still up to you to make sure they're relevant).

Running the documentation again gives:

![advanced roxygen doc example](/advanced-roxygen-doc-example.png)

You can always document more but I tend to keep it to a minimum. Everything you document can also get out of date again so it's good to strike the right balance.

Another benefit of using the `roxygen` comment is we get a warning less now when running `devtools::check()`.

## Summary

Documenting using `roxygen2` is quite easy. All you need to know is some basic tags and the `devtools::document()` function takes care of the rest. For this reproducible analysis example I haven't gone through the effort of documenting every function available. The functions are basic and are already covered with tests so in a way they're self-documenting. However, if the project would be more complex I'd certainly make sure to document every function available as a best practice.
