---
title: 'Cypress commands: selecting elements'
date: 2019-10-17T09:38:59+02:00
draft: false
---

In this post and the following one I'll go over the most basic Cypress commands which are:

- selecting elements
- act on these elements

Selections are discussed in this post, I keep the action part for the next post.

Examples are based on the [Cypress kitchen sink example](https://github.com/cypress-io/cypress-example-kitchensink) site(often slightly modified to make the point more clear in context of this post). This example app covers all existing Cypress commands.

Why start a series of testing articles with an article about an E2E testing framework? Isn't that completely against [the logic of the test pyramid](https://martinfowler.com/articles/practical-test-pyramid.html#TheImportanceOftestAutomation) where you'd try to have as many unit tests as possible and try to limit the number of E2E tests? As stated by Martin Fowler:

_Write lots of small and fast unit tests. Write some more coarse-grained tests and very few high-level tests that test your application from end to end._

However, that's only one point of view. Another strategy could be to do the [exact opposite](https://www.cypress.io/blog/2019/10/10/guest-post-new-to-front-end-testing-start-from-the-top-of-the-pyramid/). Advantages of this strategy are:

- you use a browser just like a user would
- your tests go through your app exactly as how a user would go through it
- it's easy cover a large part of the app with a limited number of tests

This post states nothing shocking but:

- it gave me a chance to go a bit more in depth as to how the Cypress basics work
- it's kind of a memory aid to myself
- links the Cypress documentation with some Devtools and CSS selection basics

## Querying

Querying in Cypress is either done based on:

- a css selector using `cy.get()`
- text using `cy.contains()`

99% of the times you will limit yourself to css selectors. Text can change making tests brittle. The `cy.get()` command may seem very clear on the outside but has a lot of interesting features.

### Waiting

One of the core features of Cypress is it waits for "key moments" of your application state to be reached. As long as these key moments have not been reached Cypress will [keep waiting](https://docs.cypress.io/guides/overview/key-differences.html#Flake-resistant). There no need for explicit waits which would make your tests flaky. How would you determine the correct waiting time anyway? Maybe the tests are ran on a slow machine, a slow network, ...

You can see how important this is by yourself. By default the timeout `cy.get()` uses is 4 seconds. If trying to run our tests without doing a `cy.visit()` first (so nothing is loaded) you'll notice the test won't fail immediately. It will keep on trying during the timeout period.

{{< highlight js >}}
// cy.visit('http://localhost:8080/commands/querying')
cy.get('#query-btn').should('contain', 'Button')
{{< /highlight >}}

![4s timeout](/cypress-4s-get-timeout.png)

And we can override the timeout to see the effect:

{{< highlight js >}}
// cy.visit('http://localhost:8080/commands/querying')
cy.get('#query-btn', {timeout:10000}).should('contain', 'Button')
{{< /highlight >}}

![4s timeout](/cypress-10s-get-timeout.png)

Note as well the timeout we get isn't exactly 4 or 10 seconds but just a bit tiny bit more. This drives home another point: Cypress runs commands **inside** the browser meaning there's [almost no lag](https://docs.cypress.io/guides/overview/key-differences.html#Flake-resistant).

Small note, `cy.get()` only checks the existence of a DOM element. This does not mean the element is rendered as you'd expect it to with all CSS applied.

### DevTools

Another interesting aspect of `get()` is it yields a jQuery object. A jQuery object is an [array-like](https://stackoverflow.com/questions/6445180/what-is-a-jquery-object) object but not exactly an array. Methods which work fine on arrays won't work for these jQuery objects. The jQuery object contains a lot of useful information you can use to write better tests. You can see for yourself if you open the [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/console/utilities) and go to the Console tab. If you click on any `cy.get()` command, a small summary of the contents of the jQuery object will be printed to the console. We want to further investigate this element so let's give it a name:

![Test button](/create-test-button-console.png)

A reference of the syntax used in the DevTools console can be found [here](https://developers.google.com/web/tools/chrome-devtools/console/utilities). Pay attention, this is no [jQuery syntax](https://api.jquery.com) so you can only run these DevTools command in DevTools itself.

Since we now have a reference to our jQuery object we can start playing around with it. Most interesting are its attributes and the css applied. Getting these values is quite simple:

![Test button](/get-attr-and-css-console.png)

However, you can go further and modify these values yourself. This way you can do some manual exploratory testing within automated tests. It's like having a REPL with your whole browser evaluating. As an example let's say we want to be sure if the href value of a hyperlink element is changed you're directed to another url (bit contrived but that's besides the point).

![Test button](/set-attr-console.png)

## Traversal

In essence just knowing the `cy.get()` command is enough to cater to all your selection needs. However, Cypress provides certain helper traversal commands to make it easier to select elements. I'll go over the main traversal commands leaving some which are rarely used aside. If an appropriate css selector [combinator](https://www.w3schools.com/css/css_combinators.asp) exists it's mentioned as well.

### children()

Does exactly as it says, gives the children of the selected element.

Its CSS selection equivalent is `>` so those are the same:

![Traversal children](/traversal-children.png)

Interestingly enough, you can filter the children selected if you pass a selector to `children()` (for example if you want to select only active children elements).

### closest()

This command traverses up the DOM tree. It stops looking further the moment it finds its match (hence closest).

![Closest HTML example](/closest-html-example.png)

Going up from the span element there are 2 div elements. Since we use `closest()` we expect it to match with the div with class well and not the one with class col-xs-5.

![Traversal closest](/traversal-closest.png)

This command has no CSS selection equivalent.

### eq()

This command get an element at an index. Its CSS selection equivalent is `nth-child()`.

![Traversal eq](/traversal-eq.png)

Pay attention: the counting in Cypress and in the CSS selection differs by 1. `nth-child()` starts at 1, `eq() starts at 0`.

### find()

This command get the descendants of a selector. The name is somewhat confusing since it doesn't refer to descendants at all. The CSS selection equivalent just uses a space.

![Traversal find](/traversal-find.png)

### next()

`next()` get the immediate next sibling of each DOM element. It's quite helpful to make assertions on several elements by just going through them one by one. CSS selection uses `+` to accomplish this. There are quite a lot of variations on this command (`nextAll()`, `nextUntil()`, ...).

![Traversal next](/traversal-next.png)

As you can see for each extra call to next you have to add an element to the CSS selector. Conceptually a call to `next()` is more obvious here.

### parent()

It's [not possible](<(https://stackoverflow.com/questions/22811495/is-parent-selector-available-in-css4)>) using only CSS4 selectors to get the parent of an element. The only possibility is using the Cypress `parent()` command.

![Traversal parent](/traversal-parent.png)

### siblings()

The last command to be discussed here is `siblings()`. Its CSS selector is `~`.

![Traversal siblings](/traversal-siblings.png)

Quite useful in a lot of cases and a straightforward command.

## Summary

Selecting elements in Cypress is not too hard. You can get by on just using `cy.get()` and some basic knowledge of CSS selectors. However, it doesn't hurt to know the jQuery and DevTools APIs somewhat to make your life easier.

If learning the traversal commands is worth the effort could be up for debate:

- in most cases using CSS selectors instead of traversal commands is easier
  - you only have to remember a single API (the CSS selectors one)
  - most CSS combinators are shorter than their Cypress equivalent
- some selections like getting the parent are close to impossible with css selectors so you need a Cypress traversal command.

But then again, if it's impossible with css selectors, maybe [you shouldn't do it](https://stackoverflow.com/questions/22811495/is-parent-selector-available-in-css4):

_Supporting it is giving people a whole lot of rope to hang themselves with._

```_

```
