---
title: "Visualize average goals per game in Processing"
date: 2019-11-26T09:36:05+01:00
draft: false
---

In a [previous analysis](https://rpubs.com/isaacverm/scoresJpl) I made a graph showing the average number of home and away goals by season. For some time I've been playing around with [Processing](https://processing.org/) but never focused on finishing something. Might be good not to focus on building something massive from the start so a graph seems relatively self-contained.

## Parts of the graph

A good place to see what we need to develop is the original [plotting function](https://github.com/IsaacVerm/scoresJpl/blob/master/R/visualize-scores.R) in `ggplot2`. The graph looks like this:

![ggplot average goals per season](/ggplot-average-goals-per-season.png)

We can see the graph has these elements:

- season on x axis
- average number of scored goals by game on y axis
- axes labels
- title
- legend
- bars (separate bars for home and away)

Some extras will be implemented:

- we'll have to make sure the graph is robust to data changes
- colours should match the colour scheme used in `ggplot2`

I won't implement the graph all at once. It seems a good idea to work from the inside out starting with the bars, then adding the x and y axes and finishing with labels, title and legend.

## Getting the data into Processing

### Average goals by season dataset

Before drawing anything we need to decide how to get the data into Processing. The [data chapter](https://processing.org/tutorials/data/) of the _Learning Processing_ book has a good explanation. The basic data format is just a `String`. This naturally leads to putting the data in csv format.

The dataset for the average goals by season already existed but still had to be written to file. While at it I made some slight modifications since the variable `goals_home_or_away` is a bit too long and reminded me a bit too much of a particular tv series. So now the [dataset](https://github.com/IsaacVerm/scoresJpl/blob/master/data/long_average_goals_by_season.csv) contains these variables:

- season
- [venue](https://english.stackexchange.com/questions/187032/is-there-a-word-for-the-status-of-a-team-being-home-or-away)
- goals

Venue refers to if a game is played at home or away.

### Loading the data

Loading the data consists of 2 steps:

- putting the data in the `/data` folder of the sketch
- reading the data

Reading the data can be done in 2 ways:

- we can either treat it as just plain text and use the `loadStrings()` function
- or we can leverage it's a csv by using the more specific `loadTable()` function

If the data is loaded as text, you get an array of which each element is a row. The first element are the headers and the rest are data values. Working with the data using the [Table](https://processing.org/reference/Table.html) class makes things way simpler. This class has convenient methods like `getStringColumn()` so you don't have to loop over all the rows each time you want to select a column. The example below shows how the data is first loaded as a table and then the season column is selected.

```
table = loadTable("average_goals_by_season.csv", "header");

String[] season = table.getStringColumn("season");
```

The first issue arising is we have both home and away goals in a single column. While implementing the function `getHomeGoals` to split the home and away goals, I ran into a slippery problem. It's [not possible](https://processing.org/reference/String_equals_.html) in Processing to compare 2 strings with `==`.

## The bars

I'm not the first one to implement a bar chart in Processing although I didn't really find a good tutorial making one. [This sketch](https://www.openprocessing.org/sketch/431/#) has some comments showing what issues might arise.

### First attempt

The main function used for bars is `rect()` for drawing rectangles. A [first implementation](https://github.com/IsaacVerm/processing-plot-jpl/blob/c312dc845457e23057b545ad7046aeac9c42a011/plot_average_goals_by_season/plot_average_goals_by_season.pde) just looping over the goals and plotting them as bars looks like this:

![basic bar chart](/basic-bars-average-goals-by-season.png)

In a sense mission accomplished since these are clearly bars but somewhat underwhelming in other departments:

- the bars are dripping from the ceiling while they should be rising from the bottom up
- whitespace to the right side of the graph
- they're just not white and not filled with any other color

### Turn the bars upside down

By default the [Processing coordinate system](https://processing.org/tutorials/drawing/) has the (0,0) point in the top left corner. In order to fix the issue, we have to base ourselves on the window height. This is what `drawHomeGoals` looks like now:

```
void drawHomeGoals(float[] home_goals) {
  // define as 2 corners instead of corner + width/height
  rectMode(CORNERS);

  // calculate variables needed for calculation corners
  float bar_width = width / (home_goals.length * 3);
  float max_home_goals = max(home_goals);
  float zone_width = 3 * bar_width;
  float y_margin = 0.1;

  // plot bars
  for (int i = 0; i < home_goals.length; i++) {
    float x_corner_one = i * zone_width;
    float y_corner_one = height * (1 - y_margin);
    float x_corner_two = i * zone_width + bar_width;
    float y_corner_two = height - home_goals[i] * (height / ceil(max_home_goals));

    rect(x_corner_one,
         y_corner_one,
         x_corner_two,
         y_corner_two);
  }
}

```

A small sketch makes the function easier to understand. In general [sketching on paper is a good idea](https://processing.org/tutorials/anatomy/) to get an idea of where you want to end up.

![sketch average goals per season](/sketch-average-goals-per-season.png)

We define a zone as a combination of 3 elements:

- bar for home goals
- bar for away goals
- empty space between bars

In comparison to the original plot the empty space is a bit larger. By default I set it as the width of 1 bar. Also these bars don't start at the edges of the window, but a small margin is provided. Later on the labels and ticks for the x axes will be placed here.

The calculation of the y position of the opposite corner of a bar is the most convoluted part. You have to scale the number of goals somehow. The maximum number of average home goals is a bit less than 2. Translated directly this would mean 2 pixels. This is negligible in a normal window size of for example 500 pixels. That's why we multiply by `height / ceil(max_home_goals)`. Taking the ceiling of the maximum home goals guarantees the bars won't exceed the height of the window.

Changing the `rectMode` to `CORNERS` enables us to draw in other directions than just down. In `rectMode` the first 2 arguments are the x and y position of one corner , the next 2 arguments are the x and y position of the opposite corner.

Note the `size()` function has the side effect of automatically creating `width` and `height` variables. Since a sketch is only valid if the size of the window is defined, we can assume `width` and `height` always to be defined. There's no need to specify them as arguments to the `drawHomeGoals`. Originally I wasn't aware of these `width` and `height` variables so I tried to do something like this:

```
int window_width = 500;
int window_height = 500;

void setup() {
  size(window_width, window_height);
```

But then you get an error saying Processing could not determine the size of your sketch.

Now the graph looks like this:

![bars bottom up](/bars-bottom-up-average-goals-by-season.png)

The bars are now neatly arranged at the bottom of the screen, but the away goals bars are still missing.
