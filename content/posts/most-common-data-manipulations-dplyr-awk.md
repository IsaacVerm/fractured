---
title: 'Most common data manipulations implemented in Awk and dplyr'
date: 2019-10-04T14:51:44+02:00
draft: false
---

[R cheatsheets](https://rstudio.com/resources/cheatsheets/)

## Goal of this post

We compare the [most common data manipulations](https://dplyr.tidyverse.org/) between `dplyr` and `awk` in terms of functionality.

As explained perfectly in the [tidyverse article](https://dplyr.tidyverse.org/) there are 5 types of common data manipulations:

- filter
- select
- mutate
- summarise
- arrange

I go over each of these manipulations and show using examples in `dplyr` and `awk` how to implement them.

Some aspects of these data manipulations are not covered in this article:

- speed (the data we used was pretty small anyway so speed mattered little)
- reading the data

## Example data

As an example I wanted to use a csv which wasn't too small. A csv because both `R` and `awk` are good at manipulating text, not too small so I could use some interesting examples instead of purely theoretical ones.

In the end I chose to use the [Denver crime dataset](https://www.kaggle.com/paultimothymooney/denver-crime-data).

## Basic explanation of awk and dplyr

I've got no intention to go over all the specifics of each but just want to outline the basic differences between using `awk` and `R`.

### dplyr

`dplyr` is an R package based on a grammar of data manipulations. The grammar compromises several verbs (select, filter, ...) which can be combined to solve most data issues. For ease of use these "verbs" can be piped (just as you would do in the command line).

### awk

`awk` reads text files line by line. When a line matches one of the patterns, `awk` performs specified actions on that line. `awk`continues to process input lines in this way until it reaches the end of the input files. Each line is split into fields.

## Setup

The following versions were used:

- R: 3.6.1
- dplyr: 0.8.3
- awk: 20070501 (Mac version)

R can be installed from the [CRAN site](https://cran.r-project.org/). After installing R you have to install the `tidyverse` packages by running the following in a R console:

```
install.packages("tidyverse")
```

For most Linux distributions `awk` is installed by default.

## Filter

Onto the data manipulations themselves now. Filtering means keeping only some records based on their values. Let's say we're interesting in [loitering](https://www.reddit.com/r/AskAnAmerican/comments/4a9x3l/what_is_loitering_and_why_is_it_illegal/).

### dplyr

```
crimes %>%
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

First thing do is find out the field index of the OFFENSE_TYPE_ID field. An easy way to this in bash without using awk (I slightly adapted this [example](https://www.biostars.org/p/248049/)):

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

Selecting means you want to pick some fields. We want to select the OFFENSE_TYPE_ID and the NEIGHBORHOOD_ID to get a quick idea of what neighborhoods have what kind of crime.

### dplyr

```
crimes %>%
  select(c("OFFENSE_TYPE_ID","NEIGHBORHOOD_ID"))
```

### awk

To find out what the field indices are we have to slightly adopt the command used above. Instead of `grep NEIGHBORHOOD_ID` at the end, we can use
`grep 'NEIGHBORHOOD_ID\|OFFENSE_TYPE_ID'`. Note the single quotes and escaping of the |. Selecting the fields is as simple as printing them:

```
awk -F, '{print $5,$17}' denver-crimes.csv
```

On a side note, it's quite straightforward to combine this with the filtering we did before:

```
awk -F, '$5~/loitering/ { print $5,$17 }' denver-crimes.csv
```

## Mutate

Mutating is adding new fields which are functions of already existing fields. A lot of the fields in the example data are related to timestamps but since timestamps are often trouble I avoid them in this post. Working with dates would probably need a post on itself to cover that material thoroughly.

As an alternative example I'll create a new field `meridiem` saying either if the crime occured before or after noon. The timestamps are formatted as `6/15/2016 11:31:00 PM` so we have to extract the last 2 characters of the timestamp.

### dplyr

```
crimes %>%
  mutate(meridiem = str_sub(FIRST_OCCURRENCE_DATE, -2,-1))
```

### awk

awk has a helper function called substr which does what we want.

```
awk -F, '{print substr($7,length($7)-2,length($7))}' denver-crimes.csv
```

There are [plenty of other options](https://unix.stackexchange.com/questions/163481/a-command-to-print-only-last-3-characters-of-a-string) like using cut (a bit dangerous because based on bytes instead of characters) or grep with the -o flag (means it only returns you what matches).

## summarise

Summarising is the practice of reducing multiple values to a single one. This is especially powerful in combination with grouping.

### dplyr

```
crimes %>%
  group_by(OFFENSE_TYPE_ID) %>%
  summarise(total = n())
```

### awk

Creating summaries in awk requires knowledge of:

- [associative arrays](https://www.gnu.org/software/gawk/manual/gawk.html#Array-Basics)
- [END special pattern](https://www.gnu.org/software/gawk/manual/gawk.html#index-END-pattern)

What we want to do is:

- go over each line and create a summary (associative array)
- print the summary (END special pattern)

As an example we'll calculate the number of crimes per crime category:

```
 awk -F, '{summary[$5]++} END {for (offence in summary) {print offence,summary[offence]}}' denver-crimes.csv
```

We loop over each offence and add 1 to the array value for that offence if it occurs in the line. After our program has looped over all the lines of offences it prints the summary (the END pattern is executed after all the lines have been looped over).

Note there's no need to say something like `summary[$5] = 0` in a BEGIN special pattern. The value of an array is instantiated by default at 0 which is exactly what we want for a total.

## arrange

In the last example we created a summary but it's not ordered. The first offence found in the `denver-crimes.csv` will also be the first offence in the summary table. It's more interesting to have the same summary with the most frequent offences at the top.

### dplyr

```
crimes %>%
  group_by(OFFENSE_TYPE_ID) %>%
  summarise(total = n()) %>%
  arrange(desc(total))
```

### awk

Using just awk for this is a bit [convoluted](https://stackoverflow.com/questions/5342782/sort-associative-array-with-awk/29005234) so we just stick to using `sort`:

```
 awk -F, '{summary[$5]++} END {for (offence in summary) {print offence summary[offence]}}' denver-crimes.csv |
```

We have to print using printf to artificially insert a comma which can be used by sort. `-t` specifies the separator to be used (the comma we inserted with the `printf`), `-k` defines the sorting field, `-n` indicates we want to sort numerically and `-r` finally makes sure we sort descending instead of ascending.

If we run the above with `head -5` to only get the top 5 of most frequent offences we get:

```
"traffic-accident",94088
"traffic-accident-hit-and-run",36660
"traf-other",33403
"theft-items-from-vehicle",27987
"theft-of-motor-vehicle",27002
```

So most "offences" are actually traffic accidents. With a minimum of code we discovered something interesting.

## Summary

`awk` has its place in data manipulation. I hope I was able to show you can do most of the basic data analysis steps from the command line.
