---
title: 'Count the smiley faces'
date: 2019-11-07T15:17:02+01:00
draft: false
---

## Problem

Create a function [counting how many valid smileys there are in an array](https://www.codewars.com/kata/583203e6eb35d7980400002a/train/javascript).

## Solution

The solution seems to need these functions:

- a function to split a smiley in its characters
- some functions to check if eyes, mouth or nose are valid

Then loop over all smileys and split/check each of them.

<script src="https://embed.runkit.com" data-element-id="my-element"></script>

<!-- anywhere else on your page -->
<div id="my-element">function countSmileys(arr) {
    splitSmiley = smiley => smiley.split("")

    checkEyes = splittedSmiley => {
        validEyes = [':',';']
        eye = splittedSmiley[0]
        return validEyes.includes(eye)
    }

    checkNose = splittedSmiley => {
        validNoses = ['-', '~']
        nose = splittedSmiley[1]
        return validNoses.includes(nose)
    }

    checkMouth = splittedSmiley => {
        validMouths = [')','D']
        mouth = splittedSmiley[splittedSmiley.length - 1]
        return validMouths.includes(mouth)
    }

    if (arr.length == 0) {
        return 0
    }

    let counterSmileys = 0

    arr.forEach(smiley => {
        const splittedSmiley = splitSmiley(smiley)
        const eyesOk = checkEyes(splittedSmiley)
        const mouthOk = checkMouth(splittedSmiley)
        let noseOk = true // if no nose it's ok

        if (splittedSmiley.length === 3) {
            // 3 means eyes, nose and mouth
            noseOk = checkNose(splittedSmiley)
        }

        if (eyesOk && noseOk && mouthOk) {
            counterSmileys += 1
        }
    })

    return counterSmileys

}

// test
const smileys = [';]', ':[', ';*', ':$', ';-D']
countSmileys(smileys)</div>

### splitSmiley

```
splitSmiley = smiley => smiley.split("")
```

If the separator is `""` the string is converted to [an array of each of its characters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split).

### checkEyes

```
checkEyes = splittedSmiley => {
        validEyes = [':',';']
        eye = splittedSmiley[0]
        return validEyes.includes(eye)
    }
```

Note you do need an explicit return keyword.

Make sure you understand the `in` [operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/in). For arrays you must specify the index number instead of the index value which might be confusing if you're used to other languages like Python or R. So this:

```
eyes in validEyes
```

would not work because it will return `false`. You have to use the `includes` method of `Array`.

### checkMouth

```
checkMouth = splittedSmiley => {
        validMouths = [')','D']
        mouth = splittedSmiley[splittedSmiley.length - 1]
        return validMouths.includes(mouth)
    }
```

At first I used here but `pop()` [changes the array its called upon](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/pop). This results in bugs difficult to find.

### countSmileys

```
let counterSmileys = 0

    arr.forEach(smiley => {
        const splittedSmiley = splitSmiley(smiley)
        const eyesOk = checkEyes(splittedSmiley)
        const mouthOk = checkMouth(splittedSmiley)
        let noseOk = true // if no nose it's ok

        if (splittedSmiley.length === 3) {
            // 3 means eyes, nose and mouth
            noseOk = checkNose(splittedSmiley)
        }

        if (eyesOk && noseOk && mouthOk) {
            counterSmileys += 1
        }
    })
```

A running counter for the number of smileys. Could have put these `if` statements in seperate functions to make things more clear.

`counterSmileys` is a `let` because it can change, if not I would have used `const`.

In the end it was too bad the tests on Codewars kept failing and I couldn't see which tests failed exactly. This made it impossible to debug.

## Alternative solutions

From the solutions with best practices [this one](https://www.codewars.com/kata/reviews/583203efeb35d7980400002c/groups/5a7543187e3b173d76000aa7) was pretty readable. He created a set of rules as separate functions feeding them into a `filter` function like this:

```
const countSmileys = smileys =>
  smileys.filter(smiley =>
    smileyIsValid(smiley) &&
    smileyHasValidEye(smiley) &&
    smileyHasValidNose(smiley) &&
    smileyHasValidMouth(smiley)
  ).length
```

It almost reads like plain English.

## Concepts

[Functions in Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions) can be defined in multiple ways:

- as a regular function
- as an arrow function

Regular function:

```
function () {}
```

Arrow function:

```
() => {}
```

Compared to a regular function an arrow function:

- has no `this` (so no separate [execution context](https://codeburst.io/all-about-this-and-new-keywords-in-javascript-38039f71780c))
- is just shorter

At first sight the syntax of the arrow function seems [almost the same](https://zendev.com/2018/10/01/javascript-arrow-functions-how-why-when.html) as for a regular function. But there are shortcuts if:

- the function body is a single expression
- there's only a single argument

In this example I also took advantage of the scoping rules by defining helper functions like `splitSmiley` within the overarching `countSmileys` function:

_The nested (inner) function is private to its containing (outer) function._
