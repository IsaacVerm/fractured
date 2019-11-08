---
title: 'Consonant value'
date: 2019-11-07T20:45:56+01:00
draft: false
---

## Problem

Calculate the word value of any word. Word value is defined as the sum of the values of the characters of any consecutive consonant string.

## Solution

Steps:

- split word in substrings of consonants
- value each letter in a substring
- sum the values in a substring
- find substring with highest sum

<script src="https://embed.runkit.com" data-element-id="my-element"></script>

<div id="my-element">function solve(s) {
    splitCons = str => str.split(/a|e|i|o|u/).filter(subStr => subStr != "")
    valueLetters = str => str.split("").map(char => char.charCodeAt(0) - 96)
    sum = arrNumbers => arrNumbers.reduce((a,b) => a + b)
    highest = arrNumbers => Math.max(...arrNumbers)

    const splittedString = splitCons(s)
    const sumValues = splittedString.map(subStr => sum(valueLetters(subStr)))
    const highestValue = highest(sumValues)
    return highestValue

};</div>

### split word in substrings of consonants

```
splitCons = str => str.split(/a|e|i|o|u/).filter(subStr => subStr != "")
```

`split()` doesn't only accept string values but also regular expressions.

A small bug occured when there's 2 consecutive vowels. In that case you'll end up with an empty string in your array of strings. Hence the `filter()`.

### value each letter in a substring

```
valueLetters = str => str.split("").map(char => char.charCodeAt(0) - 96)
```

Each character corresponds to a number in the [UTF-16](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt). You can use this to convert [letters to numbers](https://stackoverflow.com/questions/22624379/how-to-convert-letters-to-numbers-with-javascript).

### sum the values in a substring

```
sum = arrNumbers => arrNumbers.reduce((a,b) => a + b)
```

Summing can be [easily done](https://medium.com/@chrisburgin95/rewriting-javascript-sum-an-array-dbf838996ed0) with `reduce()`. You provide `reduce()` with a function which has an accumulator and a current value as arguments.

### highest

You could try to find the highest value with a combination of `reduce()` and `Math.max()`. However, `Math.max()` also accepts [rest parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters) as shown in the [example](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/max).

## Alternative solutions

Most solution seem to have followed the split, value and sum pattern. Also most used `reduce()` for finding the maximal value but I think `Math.max(...)` is nicer.

## Open questions

Is it a good idea to compose these functions in a `solve()` function?
