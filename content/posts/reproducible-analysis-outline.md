---
title: 'Reproducible analysis: outline'
date: 2019-10-31T11:31:43+01:00
draft: false
---

## Motivation

[Last analysis post I made](https://isaacverm.github.io/posts/mps-commission-attendances/) I hit quite a few stumbling blocks making the analysis take longer than expected. When reflecting on why this was the case I realized I hadn't taken reproducibility into account. I made the post in the span of a week and already I realized parts were not documented well enough, not tested well enough, code versions weren't tracked as they should, ... All this can be put under the banner of reproducibility.

Of course, [no man is an island](https://web.cs.dal.ca/~johnston/poetry/island.html), and I wasn't the first one to confront these issues. Jon Zelner has an excellent post series on exactly [this](http://www.jonzelner.net/statistics/make/docker/reproducibility/2016/05/31/reproducibility-pt-1/). His starting point is a more scientific one than I wish to entertain but the practical elaboration is still very useful (and he has an inspiring writing style). My own series is inspired by his but differs in important aspects. I plan to take on more topics (at least something about testing and about packages seems essential) and offer more examples (if my acts match my ambition is to be seen).

The end goal is to have full control over the analysis. Rerunning with different data, rerunning with different parameters, modifying whatever without having the feelings your hands are cut off because some decision down the line has locked you in. Anyone should be able to rerun the analysis without knowing every intricate detail of the analysis. The way to do it is to remove friction. Make it as easy as possible (for yourself, for anyone else) to do things. Automation can have an added value if implemented from the beginning of your analysis. A good workflow is the basis of everything else.

## Code

The basic analysis will be done in R (since I'm most familiar with the `tidyverse`) but for any other topics I use whatever suits the needs at that moment.

Code can be found in [this GitHub repository](https://github.com/IsaacVerm/reproducible-analysis).

## Example

### function for each step

The basic example used has a function for [each step in the data analysis cycle](https://r4ds.had.co.nz/explore-intro.html). So in total there are 6 steps:

- import
- tidy
- transform
- visualise
- model
- communicate

Some steps deserve some extra explanation. Tidying and transforming are distinct steps. Tidying means you put your data in tidy format. Any data obeying the following rules is considered tidy:

- each variable is a column
- each observation is a row
- each value is a cell

Sounds abstract but the [examples in the R for Data Science](https://r4ds.had.co.nz/tidy-data.html#tidy-data-1) book make clear how data can be (mis)formatted in lots of ways.

Transforming on the other hand means manipulating the data until it suits your needs. You add columns, rename columns, sort records, ... Communicating means creating a report. This as well can be automated (using [literate programming](https://en.wikipedia.org/wiki/Literate_programming) which will be covered in one of the posts later on).

### built-in dataset

To not overcomplicate things I decided to work with one of the [R built-in datasets](https://stat.ethz.ch/R-manual/R-devel/library/datasets/html/00Index.html). No package has to be loaded to have access to this data. Didn't want to go for a dataset too classic like `Titanic` so instead opted for the `nottem` [dataset](https://stat.ethz.ch/R-manual/R-devel/library/datasets/html/nottem.html). This dataset contains the average air temperature at Nottingham from 1920-1939. I opted for a dataset which has a few variables but not too many (in this case month, year and temperature). Also the dataset is not tidy by default: the rows are the years and the columns are the months.

## Topics covered

- testing
- documentation
- package development
- literate programming
- command line
- orchestrating
- environment
- version control
- continuous integration

Not sure I'll be able to cover all these topics but the order above is roughly the order in which I plan to tackle each topic. After explaining the basic example the first topic is testing. Before we do all kinds of black magic with the code it makes sense to see if it actually does what it has to do (even though the example is extremely simple). Documentation goes hand in hand with testing (because tests are in way just a type of documentation) so will be covered directly afterwards.

After that package development has absolute priority since it impacts all later choices (folder structure, integration with tests, ...). Then it's on to the command line. This is a necessary zwischenzug to be able to continue with the orchestration part (since you need something to well... orchestrate). The literature programming part following explains how to report.

Now it's onto more technical stuff with the trinity of environment, version control and continuous integration. Environment is all about getting the right versions of the software we use, version control helps us keep track of changes and continuous integration means previous parts like testing are executed automatically regularly.

So we end up with 9 topics which can be divided in 3 main parts:

- a basic program
  - testing
  - documentation
  - package development
  - literate programming
- handling complexity
  - command line
  - orchestrating
- consistency
  - environment
  - version control
  - continuous integration

It's clear the code will be refactored multiple times. I can almost guarantee following the posts chronologically won't be the fastest way to solve these issues. Pedagogical concerns trump efficiency here.

The exact contents of some of these topics might still be unclear but things will get more clear in the posts themselves. In addition, although I have a vague idea what a topic should cover, I'd like to reserve the space for myself to improvise.

## Structure topic

Since I noticed writing a post can drag on if not limiting yourself in any way I decided to use a more or less fixed template for each post in this series:

- explanation issue at hand
- solution chosen (and possible alternatives if any)
- general explanation of the solution
- examples

I put it as more or less fixed because in the end it's the subject itself dictating what's needed. But in any case I'll try to use this structure as a guideline. I'd like the posts to be short but also exhausitive. I'll try to justify whatever I do as much as possible backing it up with references to manuals, official documentation, other tutorials, ...

## What's next

A general plan is made, now it's time to make it a reality. Next post will be explaining the example used.