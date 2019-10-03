---
title: 'Higher Order Procedures'
date: 2019-10-03T15:28:01+02:00
draft: false
---

## List

higher order procedure

goal higher order procedure

higher order procedure template

higher order procedure sum template

anonymous procedure

what define does more than anonymous procedure

local variables can be created in multiple ways

## Explanation

### Higher order procedure

procedure manipulating another procedure

### Goal higher order procedure

Abstract: think about the pattern instead of the specifics.

### Higher order procedure template

```
(define (procedure-name formal-parameters)
  (if (end condition in terms of the arguments)
      finish
      do something with term and next
```

### Higher order procedure sum template

```
(define (sum term a next b)
  (if (> a b)
      0
      (+ (term a)
         (sum term (next a) next b))))
```

### Anonymous procedure

Procedure without a name (called lambda in Scheme).

### define does double work

- creates a procedure
- gives that procedure a name

## Gotchas

Make sure to do something with an argument so the end condition is ever reached.
