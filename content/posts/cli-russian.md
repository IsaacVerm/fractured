---
title: 'Creating a simple CLI with Click to learn Russian (part 1)'
date: 2019-10-15T13:53:14+02:00
draft: false
---

## Goal

### Unix philosophy

In line with the [Unix philosophy](https://homepage.cs.uri.edu/~thenry/resources/unix_art/ch01s06.html) I wanted to build a small system which is as user-friendly as possible (even though I might be the only user). I chose a system which would help me with a personal goal namely learning Russian.

### Learning Russian

Recently I [scraped](https://github.com/IsaacVerm/scrape-master-russian) example phrases for the [most common Russian words](http://masterrussian.com/vocabulary/most_common_words.htm). Based on [Zipf's law](https://en.wikipedia.org/wiki/Zipf%27s_law) it makes sense to really focus on the most common words since _the frequency of any word is inversely proportional to its rank in the frequency table_. Just learning vocabulary is kind of sterile so I spice it up by learning some basic phrases.

Learning those phrases involves studying them which means:

- annotating with the function of each word (cases)
- finding the root word of each word
- annotating with etymology to help remember the translation
- annotating with similar words
- annotating with related terms

Since each of the above has to be done a lot of times it makes sense to create a small CLI to interact with the phrases. For example the first phrase of our list is:

```
Мальчик и девочка играют.	A boy and a girl are playing.
```

We have some issues with the word `девочка` often confusing it with `дедушка` which means grandfather. Wouldn't it be nice to be able to type:

```
similar девочка дедушка
translation дедушка grandfather
```

Let's say the next time we go over the phrases we want to train all the similar words:

```
similar
```

Which gives a list of similar words. These are just some useful ideas for commands.

### Features

Based on the above I want the system to have the following:

- each command has a help page
- documented
- minimalist (_Make each program do one thing well_)
- defensive (no way to input wrong data)

## Setup

Setup as explained in cli-russian README [repository](https://github.com/IsaacVerm/cli-russian).

## Design

There are plenty of things we can do with the phrases but the most important thing is to be able to:

- annotate
- persist these annotations

You quickly realize annotating a word isn't as straightforward as it seems. For each type of word (noun, verb, adjective, ...) the information we want to add is different. For all of them we'll want to add the Russian word but for example gender does make sense for nouns but doesn't for verbs.

After mulling it over I ended up with multiple annotation commands. They're grouped by type of word (noun, verb, ...):

- `annotate conjunction <russian> <english>`
- `annotate preposition <russian> <english> <related case>`
- `annotate particle <russian> <english>`
- `annotate pronoun <russian> <root> <english> <case>`
- `annotate verb <russian> <root> <english> <time> <person> <number>`
- `annotate noun <russian> <root> <english> <case> <gender> <number>`
- `annotate adjective <russian> <root> <english> <case> <gender> <number>`

By article words are meant which don't change form (like и),

## Basic implementation

I used [Click](https://click.palletsprojects.com/en/7.x/) to design the command line interface.

### Initial example

Having more or less defined where we want to end up with an initial prototype we create a first implementation. If we want to implement our commands we need:

- a way to have the code interpreted as cli commands
- a way to pass arguments to our commands

Both are explained excellently in the Click [quickstart](https://click.palletsprojects.com/en/7.x/quickstart/). Click is entirely based on [decorators](https://realpython.com/primer-on-python-decorators/). There's a lot to it but in essence decorators are just functions wrapping other functions to change how these functions behave.

So this is how things look like for our first `annotate-noun` command:

```
@click.command()
@click.argument('russian')
@click.argument('root')
@click.argument('english')
@click.argument('case')
@click.argument('gender')
@click.argument('number')
def annotate_noun(russian, root, english, case, gender, number):
click.echo('Russian noun is %s' % russian)
click.echo('Based on root noun %s' % root)
click.echo('English translation of this noun is %s' % english)
click.echo('Case is %s' % case)
click.echo('Gender is %s' % gender)
click.echo('Number is %s' % number)
```

`@click.command` makes the `annotate-noun` function a command. `@click.argument` helps us pass arguments to `annotate-noun`.

If we call `annotate-noun` with `python annotate.py человеки человек person nom m sg` we get:

```
Russian noun is человеки
Based on root noun человек
English translation of this noun is person
Case is nom
Gender is m
Number is sg
```

But say we forgot we had to specify the gender and call with `python annotate.py человеки человек person nom pl`. We get:

```
Error: Missing argument "NUMBER".
```

Because all arguments are positional the error is complaining about the number while it should have complained about the gender. That's why, before moving on to the implementation itself, we'll make our command a bit more robust to user errors.

### Usability

We can improve the usability by:

- using options instead of arguments
- prompting
- validate input
- help page

[Arguments](https://click.palletsprojects.com/en/7.x/arguments) and [options](https://click.palletsprojects.com/en/7.x/options) are very much alike. However, arguments are positional and so they have to be specified by definition. Well, options are optional, no surprises there. Since I noticed prompts are only available for options there's no other choice then to use options instead of arguments.

Validation is done with choice options. You pass a list of valid values and if the input by user doesn't match any of these options an error is displayed. With some hassle it's possible to have [custom error messages](https://stackoverflow.com/questions/39596070/python-click-custom-error-message) but I won't go into that in this post.

The [help page](http://click.palletsprojects.com/en/7.x/documentation/) is built from:

- the help arguments to `click.option`
- the docstring of the function

By specifying `--help` e.g. `annotate_noun --help` you get access to the help page.

This is the result of making the code more user-friendly:

```
@click.command()
@click.option('--russian', prompt='Give the original Russian noun', help='the declined form of the noun')
@click.option('--root', prompt='Give the root', help='nominative singular form of the noun')
@click.option('--english', prompt='Give the English translation of the noun', help='English translation of the noun')
@click.option('--case', type=click.Choice(['nom', 'gen', 'dat', 'acc', 'ins', 'pre']), prompt='Specify the case of the noun', help="case can be either nom(iniative), gen(initive), dat(ive), acc(usative), ins(trumental) or pre(positional) ")
@click.option('--gender', type=click.Choice(['m', 'f', 'n']), prompt='Specify the gender of the noun', help="gender can be either m(asculine), f(eminine) or n(euter)")
@click.option('--number', type=click.Choice(['sg', 'pl']), prompt="Specify the number of the noun", help="number can be either sg (singular) or pl (plural)")
def annotate_noun(russian, root, english, case, gender, number):
"""Adds annotation for the Russian noun."""
...
```

`click.Choice` takes care of the validation, the `"""..."""` docstring is used in the help and the `prompt` argument triggers a prompt.

### Persistence

Now we have everything setup to get a clear interface, we can focus on the persistence.

This part is as simple as:

```
with open('nouns.tsv', 'a') as out: # tsvs use tabs as separators
tsv_writer = csv.writer(out, delimiter='\t')

        # write noun as row
        tsv_writer.writerow([russian, root, english, case, gender, number])
```

`with` is a Python function helping with opening and closing the file so we don't have to do it ourselves. The writer object which has a `writerow` function.

## Summary

We can now type `annonate_noun` in the command line and are prompted to fill in the required fields for annotating a noun:

```
$ annotate_noun
Give the original Russian noun: человека
Give the root: человек
Give the English translation of the noun: person
Specify the case of the noun (nom, gen, dat, acc, ins, pre): gen
Specify the gender of the noun (m, f, n): m
Specify the number of the noun (sg, pl): sg
Annotation has been added to the nouns.tsv file.
```

If we want help we can get it as well:

```
$ annotate_noun --help
Usage: annotate_noun [OPTIONS]

  Adds annotation for the Russian noun.

Options:
  --russian TEXT                  the declined form of the noun
  --root TEXT                     nominative singular form of the noun
  --english TEXT                  English translation of the noun
  --case [nom|gen|dat|acc|ins|pre]
                                  case can be either nom(iniative),
                                  gen(initive), dat(ive), acc(usative),
                                  ins(trumental) or pre(positional)
  --gender [m|f|n]                gender can be either m(asculine), f(eminine)
                                  or n(euter)
  --number [sg|pl]                number can be either sg (singular) or pl
                                  (plural)
  --help                          Show this message and exit.
```

This was just an example with a single command. However, nothing stops us from scaling it up with all the commands designed. The next part will be all about that.
