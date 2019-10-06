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

Reading the data is out of scope for the comparison.

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

Install R and the following packages:

```
install.packages("tidyverse")
```

`awk` is installed by default.

## Filter

Filtering means keeping only some records based on their values. Let's say we're interesting in [loitering](https://www.reddit.com/r/AskAnAmerican/comments/4a9x3l/what_is_loitering_and_why_is_it_illegal/).

### dplyr

```
loitering <- crimes %>%
  filter(OFFENSE_TYPE_ID == "loitering")
```

### awk

[awk](https://www.tim-dennis.com/data/tech/2016/08/09/using-awk-filter-rows.html) offers several options.

#### without specifying field

We could just use a regular expression and not specify the field. This works quite well if the value we want to filter on is unique among the whole dataset. For example if we filter on 'loitering' it's not quite likely any of the other fields would have a value loitering as well (they're mostly offence codes and geographic data):

```
awk -F, '/loitering/' denver-crimes.csv
```

Notice a normal awk program consists of a series of pattern/action pairs called rules. In the example above only the pattern `/loitering/` is specified. If no action is specified the default action is to [print the entire line](https://www.gnu.org/software/gawk/manual/gawk.html#Very-Simple).

#### field-specific

If we want to be more specific we could use the [regular expression operator ~](https://www.gnu.org/software/gawk/manual/gawk.html#Regexp-Usage) only on the field containing the OFFENSE_TYPE_ID.

First thing do is find out the field index of the OFFENSE_TYPE_ID field. An easy way to this in bash without using awk (I slightly adapted the [example](https://www.biostars.org/p/248049/)):

```
head -1 denver-crimes.csv | tr ',' '\n' | cat -n | grep "OFFENSE_TYPE_ID"
```

The command above has 4 steps:

- print the first line containing the field names (`head -1`)
- translate commas to newlines (`tr ',' '\n'`)
- print all lines (`cat`) with their line number (`-n`)
- search the line containing the field we want

The translation from comma to newline is necessary to have multiple lines for `cat`. Since now we know the index of the OFFENSE_TYPE_ID we can be more specific with our regex:

```
awk -F, '$5~/loitering/' denver-crimes.csv
```

To make things more readable we could give the field index a name with a [variable](https://www.gnu.org/software/gawk/manual/gawk.html#Using-Variables). In practice you probably won't do it for throwaway programs.

```
awk -F, '$offence_type_id~/loitering/' offence_type_id=5 denver-crimes.csv
```

## Select

We want to select the OFFENSE_TYPE_ID and the NEIGHBORHOOD_ID to get a quick idea of what neighborhoods have what kind of crime.

### dplyr

```
offense_type_by_neighborhood <- crimes %>%
  select(c("OFFENSE_TYPE_ID","NEIGHBORHOOD_ID"))
```

### awk

To find out what the field indices are we have to slightly adopt the command used above. Instead of `grep NEIGHBORHOOD_ID` at the end, we can use
`grep 'NEIGHBORHOOD_ID\|OFFENSE_TYPE_ID'`. Note the single quotes and escaping of the |. Selecting the fields is as simple as printing them:

```
awk -F, '{print $5,$17}' denver-crimes.csv
```

On a sidenote, it's quite straightforward to combine this with the filtering we did before:

```
awk -F, '$5~/loitering/ { print $5,$17 }' denver-crimes.csv
```
