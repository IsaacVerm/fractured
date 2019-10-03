---
title: 'Tips and Tricks'
date: 2019-10-03T16:08:12+02:00
draft: false
---

## Debugging

If you use `R5RS` as language, you can use the `trace` function to see the call stack.

E.g.

```
(#%require racket/trace)

(define (sum-integers a b)
  (if (> a b)
      0
      (+ a (sum-integers (+ a 1) b))))

(trace sum-integers)

(sum-integers 1 5)
```

Don't forget to specify the require because [trace] is [not included in R5D5 by default](https://stackoverflow.com/questions/12667394/require-not-working-in-dr-racket).
