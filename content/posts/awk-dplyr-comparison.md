---
title: 'Awk Dplyr Cheatsheet'
date: 2019-10-04T14:51:44+02:00
draft: false
---

[R cheatsheets](https://rstudio.com/resources/cheatsheets/)

## Goal

Compare the [most common data manipulations](https://dplyr.tidyverse.org/) between `dplyr` and `awk` in terms of:

- speed
- functionality

As explained perfectly in the dplyr article there are 5 types of common data manipulations:

- mutate
- select
- filter
- summarise
- arrange

All of these supported by grouping. Each of these explanations is explained more in detail in the relevant section.

## Test data set

Needs to be:

- csv (so it can be used easily by both R and awk)
- quite sizable

Ended up picking the [Denver crime dataset](https://www.kaggle.com/paultimothymooney/denver-crime-data).

## Software versions

R: 3.6.1 (released on 2019-07-05)
dplyr:

awk: 20070501

## How they work

### dplyr

`dplyr` is an R package.

### awk

### benchmarking

[Benchmarking article](https://www.alexejgossmann.com/benchmarking_r/)

[R advanced profiling](http://adv-r.had.co.nz/Profiling.html)

Benchmarking in R is done by using [tictoc](https://www.jumpingrivers.com/blog/timing-in-r/).

## Setup

### R

Install R and the following packages:

```
install.packages("tidyverse")
```

## Data manipulations

Reading the data is out of scope for the comparison.

### Filter

Filtering means keeping only some records based on their values. Let's say we're interesting in [loitering](https://www.reddit.com/r/AskAnAmerican/comments/4a9x3l/what_is_loitering_and_why_is_it_illegal/).

#### dplyr

```
loitering <- crimes %>%
  filter(OFFENSE_TYPE_ID == "loitering")
```

#### awk

[awk](https://www.tim-dennis.com/data/tech/2016/08/09/using-awk-filter-rows.html) offers several options.

We could just use a regular expression and not specify the column. This works quite well if the value we want to filter on is unique among the whole dataset. For example if we filter on 'loitering' there's no way any of the other fields would have a value loitering as well (they're mostly offence codes and geographic data):

```
awk -F, '/loitering/' denver-crimes.csv
```

If we want to be more specific we could use the [regular expression operator ~](https://www.gnu.org/software/gawk/manual/gawk.html#Regexp-Usage) only on the field containing the OFFENSE_TYPE_ID:

```
awk -F, '$5~/loitering/' denver-crimes.csv
```
