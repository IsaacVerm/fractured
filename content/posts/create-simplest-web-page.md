---
title: 'Create simplest web page possible'
date: 2019-09-08T16:12:09+02:00
draft: false
---

## Rationale

Everyone comes in touch with the web every day. Opening a browser is as natural as waking up. Sadly enough, way less people know how the web really works. That's a pity because once you know how it work possibilities are endless. You can start manipulating the web any way you want which is close to being a magician!

In this tutorial I'd like to show:

- web development isn't hard to learn
- and learning it is fun
- while learning you are already creating something
- and creating is fun
- so you have already double fun
- and then you can share it
- sharing is also fun

So as you can see you're set up for triple whammy of pleasure with this article.

To learn about the web I cannot recommend the [MDN documentation](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web) enough. However, I can't help but feel the minimal level required to start with that documentation might still be a bit too high for people coming from a total non-technical background. In this tutorial I'll try to bridge that gap by explaining just enough to be able to do what you want but not more.

So it's not because a concept isn't mentioned in this tutorial it doesn't exist. It's just not mentioned because it would be confusing to introduce it too early. Take this both as a challenge and a reassurance: it's easy to learn enough to do something but there's a whole world of things more to learn!

## What we'd like to make

As as example I'd like to have:

- a functioning web page
- available on the internet for everyone to access

As will be shown, both can be achieved by just a single file containing just a single line of text.

## First steps

### Holy Trinity

Starting from a blank page is always hard. Let's first have a look at what technologies we'll have to use to create our web page.

In web development there's something called the Holy Trinity (that's not an official name but it makes for a nice analogy) of HTML, Javascript and CSS:

- HTML structures content
- Javascript is used to modify that HTML
- CSS describes how the HTML content should be presented

As you may have noticed both Javascript and CSS refer to HTML. So what's HTML then?

### What's HTML?

From [the MDN documentation on HTML basics](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/HTML_basics) we learn HTML refers to elements. Elements wrapped or enclosed or in whatever constellation but in the end it's just elements. HTML elements are like atoms in the real world. They're the basic building blocks of all web pages.

So how do we know which elements we can use? There's an organisation called the [World Wide Web Consortium](https://www.w3.org/) which has standardized what elements can be used. A reference of all these elements can be found [here](https://developer.mozilla.org/en-US/docs/Web/HTML/Element).

Let's open the HTML candy shop and see what we can feast on! There are elements related to text (e.g. `<p>` for paragraphs), elements related to multimedia (e.g. `<img` for images), for forms, for buttons, whatever you can think of.

So that might make you think: if HTML is elements and elements are like atoms, could we create a web page with only HTML? Well, yes!

### Creating your first HTML page

Let's stick to text elements for now. since they're the easiest to understand. Create a file somewhere and call it `index.html`. The file can have any name but by convention the homepage is called the `index.html` file. Within this file we put a single paragraph element:

```
<p>I am a paragraph</p>
```

If we now open the `index.html` file with a browser like Chrome, you'll see the browser will display the text we want. The html file contains nothing more than a single element with some text but that's already a web page!

Interesting to note is the browser doesn't just display `<p>I am a paragraph</p>` but interprets the elements. It doesn't display the paragraph tags because it know `<p></p>` means we want to display just the text.

## Deployment

### Why a local file is not enough

Now we have a web page we're ready to share it with the world. At the moment the `index.html` file is just a file on our computer and. Since we're the only ones to have access to our computer our web page is not online yet.

If we want to share it we need a server. The server will take care of requests by browsers for our web page. The browser will ask the server to give it the web page and the server will gladly [give it to the browser](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/How_the_Web_works). We could make our own computer the server but that would mean it has to be on 24 hours a day. A better solution is to use an external server provider.

### Which server provider to use?

The most popular choice to host static HTML pages like the one we just created is to use [GitHub Pages](https://pages.github.com/). However, to use GitHub Pages you need knowledge of:

- the command line
- Git

The command line is a [text interface](https://www.codecademy.com/articles/command-line-commands) for your computer instead of the graphical user interfaces (GUI) most of us are used to in daily life (for example the Spotify app is a GUI). To use it effectively you need to know some basic commands which would be too much for a tutorial like this.

[Git](https://git-scm.com/) is a kind of computer language which helps tracking changes between files, a version control system. It can be used in a command line or GUI way but in the end the concepts needed to use it effectively would also be outside the scope of this tutorial.

Next to GitHub Pages there are other options like [surge](https://surge.sh/). Surge has as an advantage you don't need to know about version control systems like Git but you still need to know your way around the command line.

So in the end the best option for us now turned out to be [Neocities](https://neocities.org), the successor of Geocities (fittingly known for its 90s looking sites which is exactly what we're aiming for here).

To use Neocities to host your `index.html` file you need to:

- open a Neocities account
- go to the [Neocities dashboard](https://neocities.org/dashboard)
- add the `index.html` file to Home
- open your site

![Neocities dashboard](/neocities_dashboard.png)

And there it is, your first web page.

![Simplest web page example](/neocities_simplest_web_page.png)

Doesn't look like nothing much but we got what we wanted. We have a functioning web page and it's available for everyone to visit.

## What you did

### Learn HTML elements

You got to know HTML, the most fundamental building block of the web. In essence most of web development is just changing HTML elements in one way or another. The big difference between this web page and more complex ones is the way in which those elements are changed. If we want to add or remove an element we have to manually edit the file. More complex web pages use a separate programming language to do this.

But so even these dynamic sites in the end just deal with HTML elements. There can be lots of layers of complexity built on top of it but in the end the basics are easy to understand. And that's reassuring. Things get only easier from here on.

### Deployment

You learned how to share the web page you created with others on a server. Manually doing it on Neocities isn't the best way to do it but it got the trick done, the site is there and people can open it!

## What you didn't do

### Use more elements

The only element we used is the paragraph elements. There are way more elements and it's worth looking into them. The [Mozilla site](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/p) is a good place to start experimenting. No need to install anything, you can edit HTML live and immediately see the result.

### Best practices

Except for a manual check to see if our page loads we didn't test anything. However, testing can also be done using a programming language with automated tests.

We also didn't use any version control. Once your web page starts to grow it makes sense to keep track of changes in your files and upload your code to a remote server so you don't risk losing all your work.

### Dynamically manipulate elements

The Holy Trinity consists of HTML, Javascript and CSS. This tutorial didn't touch Javascript or CSS. The use of Javascript determines the difference between a static and dynamic site. The difference between static and dynamic lies in the moment the HTML is created:

- static sites generate all the HTML on the server and then hand it over to the browser
- dynamic sites change the HTML after they received it from the server

For example say you want a button on your site which remove some text when you click on it. Removing the text is changing the HTML so you need a dynamic site (with Javascript) to make this work.

## Conclusion

There are a few things to remember after this tutorial:

- HTML means elements
- your browser interprets these elements
- you need a server to make your web page available to everyone.

Everything else builds on these crucial concepts.

## What's next?

### Static site generator

You may have noticed editing HTML files yourself can be kind of tedious. Say we want 10 different pages but they're all kind of the same, only a smart part of them changes from page to page.

In that case we could use a static site generator. For example this blog is made using a static site generator called [Jekyll](https://jekyllrb.com/). What these static site generators add is they use templates. Using template we can define a blueprint for a type of page and then each individual page can just use that blueprint and add whatever is specific to that page.

### Dynamic pages

Let's say we want our site to change based on user input. For example if a button is pressed we want a piece of text to be displayed. To connect the user input to our HTML we'll have to use some kind of language. By far the most popular language to do this is Javascript.

Start off with a minimal framework like [Vue](https://vuejs.org/). This framework doesn't have a lot of bells and whistles so it's easier to grasp the basics.
