---
title: 'Exercise 1.3'
date: 2019-09-28T19:31:21+02:00
draft: false
---

{{< gist IsaacVerm 6c356929ab439536c3937db65452b5a0 >}}

## concepts used

- procedure definition
- conditional expression

## explanation concepts

### [procedure definition](https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/book-Z-H-10.html#%_sec_1.1.4)

Abstraction by which a compound operation can be given a name and then referred to as a unit.

General form:

```
(define (<name> <formal parameters>) <body>)
```

- name (symbol to be associated with the procedure definition in the environment)
- formal parameters (names used within the body of the procedure to refer to the corresponding arguments of the procedure)
- body (expression that will yield the value of the procedure application when the formal parameters are replaced by the actual arguments to which the procedure is applied)

### [conditional expression](https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/book-Z-H-4.html#%_toc_%_sec_1.1.6)

Consists of clauses which are pairs of predicates and consequent expressions.

General form:

```
(cond (<p1> <e1>)
      (<p2> <e2>)

      (<pn> <en>))
```

## Remarks

You should use `<=` or `>=` instead of `<` and `>`. If not the case where 2 or more of the numbers are equal fails.
