---
title: 'Exercise 1.5'
date: 2019-09-29T20:25:54+02:00
draft: false
---

{{< gist IsaacVerm df0b38099e97c2047fe2bf51b2d8bb8b >}}

Explanation in [blog post](https://www.lvguowei.me/post/sicp-goodness/) easier than the one the book.

Evaluation:

- normal order
  - expand and reduce
    - all functions first
    - then evaluate parameters
  - lazy
- applicative order
  - evaluate and apply
    - function first
    - then formal parameters
    - loop
  - eager

So why does the test work? p will never stop executing when evaluating since there's no stopping condition. So if we use the _evaluate and apply_ model from applicative order evaluation the first thing that will be done is evaluating the p procedure thus hanging the process.

However, if we use the _expand and reduce_ model from normal order evaluation the p procedure will never be evaluated since it doesn't meet the 0=0 condition.
