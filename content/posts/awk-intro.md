---
title: 'Awk Intro'
date: 2019-09-19T20:26:51+02:00
draft: true
---

## Why this post?

Nothing revolutionary, but good way to focus if you combine learning with writing.

## Why AWK

I love Web page is written with a capital in the GNU documentation.

### works with any other command line output

e.g. `docker images` can be piped into AWK.

### good to work with csvs

### limited scope but does it very well

```
The basic function of awk is to search files for lines (or other units of text) that contain certain patterns. When a line matches one of the patterns, awk performs specified actions on that line. awk continues to process input lines in this way until it reaches the end of the input files.
```

### simple conceptually

AWK program consists of rules. Each rule looks like this:

```
pattern { action }
pattern { action }
```

```
The input is read in units called records, and is processed by the rules of your program one record at a time.
Each record is automatically split into chunks called fields.
```

### widely available

AWK is [installed by default](https://hub.packtpub.com/awk-programming-langauge/) on most GNU/Linux distributions.

### can be used interactively

By just typing `awk 'program'` in the command line.

## Run

There are multiple ways to run awk.

Interactively

```
awk 'program'
```

In command line:

```
awk 'program' source
```

From file:

```
awk -f source
```

## Examples

### search for string and print current line

```
awk '/li/ { print $0 }' mail-list
```

`/li/` is a pattern which is a regular expression, `{ print $0 }` is the action, `mail-list` is the source.

This can be easily used to filter. The command belows gives all lines for friends.

```
awk '/ F/ { print $0 }' mail-list
```

### omit pattern or action

```
If the pattern is omitted, then the action is performed for every input line. If the action is omitted, the default action is to print all lines that match the pattern.
```

So this:

```
awk '/li/ { print $0 }' mail-list
```

could as well have been written as:

```
awk '/li/' mail-list
```

since printing is the default action.

### AWK goes line by line and rule by rule

So AWK:

- begins on the first line
- checks the pattern of the first rule
- checks the second, third,... pattern of the first rule (if they exist)
- goes to the next line if all patterns have been checked

This can by example be used to duplicate each row:

```
awk '/-/ { print $0 } /-/ { print $0 }' mail-list
```

There are 2 rules. Since the - appears on each line, each line is printed twice.

## Main concepts

### Predefined variables

There's NR (total number of input records read so far from all data files) and FNR ( the number of records that have been read so far from the current input file).

If RS (record separator) is any single character, that character separates records.

### Fields

You use a dollar sign (‘\$’) to refer to a field in an awk program, followed by the number of the field you want.

By default, fields are separated by whitespace (space, tab and newline).

The use of \$0, which looks like a reference to the “zeroth” field, is a special case: it represents the whole input record.

## Gotchas

/ for line continuation, ; to have multiple rules on the same line.

## Sources

[Grymoire](http://www.grymoire.com/Unix/Awk.html#TOC)

[GNU](https://www.gnu.org/software/gawk/manual/gawk.html)

GNU is accessible source.

## To do

- compare with `dplyr` examples
- own examples instead of GNU ones
