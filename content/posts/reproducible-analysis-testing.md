---
title: 'Reproducible analysis: testing'
date: 2019-11-02T10:45:16+01:00
draft: false
---

## Issue

When writing code we have some expectations about what it should do. We'd like it do something and, even better, do exactly what we want. Verifying this is harder than it seems. You can load a function, run it with some data and check manually if you like the results. However, this scales pretty badly as your code base grows. It becomes impossible to manually check all the details and accidents are bound to happen. Another problem is this type of tests is rather ephemeral. They're saved nowhere (except maybe in your head) so you'll have to make them again next time you want to test your code.

Those are the main reasons I started writing automated tests. But there are other benefits. If focusing on testing almost magically you write better code because you try to keep your functions as small and modular as possible. Tests also give you a goal to work towards (if writing the tests beforehand, which is a best practice but demands a level of discipline I rarely possess). This makes it especially easier to pick up code you didn't work on for some time since you get an idea of where exactly you were. Last but not least, tests document your code without having to write explicit documentation.

There is a place for a hoc tests. It never hurts to play around.

## Solution and alternatives

I chose to use the `testthat` package. However, it doesn't seem to [play well](https://github.com/r-lib/testthat/issues/659) with code not written as a package. At the moment my code is just a single `analysis.R` file which is an issue. I could have tried a workaround but this might be more trouble than it's worth. Better to postpone this testing part and [package](http://r-pkgs.had.co.nz/intro.html) our analysis first.

## How testthat works

## Examples

## Sources

http://r-pkgs.had.co.nz/tests.html
