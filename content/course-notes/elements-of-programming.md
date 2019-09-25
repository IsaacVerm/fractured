---
title: 'Elements of Programming'
date: 2019-09-25T21:40:27+02:00
draft: true
---

## elements in every language

- primitive expressions
- combination
- abstraction

## primitive expressions

- most basic entities
- either
  - primitive procedures (e.g. +)
  - primitive data (number)

## combination

build compound elements from simpler elements

## abstraction

compound elements can be named and manipulated as units

## types of elements

- procedures
- data

## benefit prefix notation

- arbitrary number of arguments
- nesting

## repl

read-evaluate-print-loop

## define in scheme

variables have values

`define` in scheme is simplest abstraction

## environment

memory keeping track of the name-object pairs

## how interpreter evaluates

Combinations:

- evaluate subexpressions
- apply procedure leftmost expressions to other subexpressions

Primitives:

- value of number is the number
- values other names are objects associated with those names in the environment

## consequence evaluation interpreter

meaningless to speak about expression without specifying its environment

## special forms

exceptions to the general evaluation rule

e.g. `(define size 2)` does not apply operator `define` to operands `size` and `2`.

special forms have their own evaluation rule

syntax: which special forms are available in programming language

## recursive

one of the steps in the rule is invoking the rule

## tree accumulation

values percolate upward in tree

## procedure definition in lisp

```
(define (<name> <formal parameters>) <body>)
```

## normal substitution model

Better explained [here](https://www.lvguowei.me/post/sicp-goodness/) than in book.

1. evaluate all functions
2. evaluate all arguments

## applicative substitution model

loop of:

1. evaluate function
2. evaluate arguments of that function
