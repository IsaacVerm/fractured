---
title: "Web app for easy access to newspapers: Django, Travis, Cypress and Heroku"
date: 2019-12-02T14:22:24+01:00
draft: false
---

## Use case

For the elderly it can be hard to use modern technology. There can be so many obstacles blocking you from accessing the content you want. I noticed my grandmother struggling and decided to make a small web app tailored to her needs. The web app had to be as simple as possible, offer no way to get lost and be accessible (large font so it's easy to read, clear and concise language, ...)

My grandmother is mainly interested in reading newspapers but the request to accept cookies which always popup did really throw her off.

![cookie prompt](/cookie-prompt.png)

Making a small web app wrapping RSS feeds from a newspaper (in this example [De Standaard](https://www.standaard.be/rssfeeds)) put the above into practice.

## Tools

The web app itself is pretty basic but I wanted to experiment with some tooling around this app:

- testing
- continuous integration
- cloud deployment

In the end I chose to build the web app with [Django](https://www.djangoproject.com/) and test with [Cypress](https://www.cypress.io/). Deployment is done on [Heroku](https://www.heroku.com/), continuous integration is handled by [Travis](https://travis-ci.org/).

### Why Django?

From the [Django main page](https://www.djangoproject.com/):

> Django is a high-level Python Web framework that encourages rapid development and clean, pragmatic design.

This is exactly what I was looking for. The idea behind this post is to focus on the tooling around the app, not the app itself. The web app serves just as a practical example. I have used Django previously and know how easy it is to get something up and running fast. Added benefit of opting for Django is it's Python and I should brush up my Python skills somewhat.

Django is batteries included so there are plugins for all the most common web development tasks. I need at least something to handle mail and a database since it's possible to save articles. Django has an [object-relational mapper](https://docs.djangoproject.com/en/3.0/topics/db/models/) which means you can specify what your database should be like in Python code. The model is the single source of truth. Django also provides [mail functionality](https://docs.djangoproject.com/en/2.2/topics/email/).

Another advantage of Django you notice when browsing through the documentation is everything is documented and it's documented well. It's always a boon not to have to figure the basics out over and over again.

For a split moment I considered going for [Ruby on Rails](https://rubyonrails.org/) as web framework since it's based on the same Model-View-Controller paradigm. However, I don't want to learn Ruby for now and just focus on Python.

### Why Cypress?

The app should be tested but since the app itself is not that complex, I didn't want to go overboard with several types of tests. Best choice in this case was to write E2E tests so when these tests pass I can be quite sure everything's fine.

I've used Cypress at work and have good experiences with it. There are numerous advantages to using Cypress but off the top my head what I liked most so far is:

- the debugger where you can see the state of the DOM at any time
- the integration with [DevTools](https://developers.google.com/web/tools/chrome-devtools) (especially handy if combined with the debugger)
- can automatically take screenshots (and videos, but never used that feature)

### Why Heroku?

Just as Django and Cypress, I also used Heroku before. There are a few [reasons not to use Heroku](https://medium.com/@brenda.clark/heroku-alternatives-top-5-picks-9095cef91d91) but non of those applied for this project. Django is [supported by Heroku](https://devcenter.heroku.com/articles/django-app-configuration), I don't need to do anything fancy and I'm still in the free tier. Since Heroku is so widely used it's easy to find quick fixes if running into any issues. Google has an alternative with [Google Cloud](https://cloud.google.com/) but I didn't really see any added benefit over using Heroku.

### Why Travis?

Travis I also used before to continuously test [R code](https://travis-ci.org/IsaacVerm/marriage) and [Flask apps](https://github.com/IsaacVerm/postcards-tests/blob/master/.travis.yml). Using Travis is quite simple, you more or less just have to make sure you have a `.travis.yml` configuration file and follow the [Travis API](https://docs.travis-ci.com/). The main alternative for Travis seems to be [CircleCI](https://circleci.com/). Travis seems to have [a bit more features](https://hackernoon.com/continuous-integration-circleci-vs-travis-ci-vs-jenkins-41a1c2bd95f5) than CircleCI but in essence they work pretty much the same way.

## Approach

I started from a [small example](https://homepage.cs.uri.edu/~thenry/resources/unix_art/ch01s07.html) which functionally didn't have a lot to it (it was just a list of articles from the RSS feed) but already implemented all the technologies I wanted to use. This turned out to be a good idea. The continuous integration offered by [Travis](https://travis-ci.com/) really added a while developing. It happened a few times I forgot to add dependencies to the `requirements.txt` file. These dependencies were installed locally so I only ran into trouble when deploying to a separate server (using [Heroku](https://www.heroku.com/)). And I only knew there was trouble thanks to the [Cypress](https://www.cypress.io/) tests. This trinity of tests, early deployment and continuous integration is something I'll certainly try to implement in future projects.

## Design

The whole application revolves around these flows:

- getting the latest articles
- seeing your saved articles
- opening an article
- saving an article
- sending an article by mail

The main action takes place in the article detail view where you can:

- see the content of an article
- save the article for later use
- send the article to someone

Because the article detail is so important it can be accessed from several other pages:

- saved articles page
- feed of articles pages

The articles saved page is displayed:

- after saving an article
- on the main page

Since this web app is all about accessibility all the pages are connected. You can never get stuck anywhere in the application.

There are also safeguards against saving multiple times. If an article is already saved, you can't save it again so there's no risk of duplicates.

## Summary

This post covered why the app was created, the tools used and how the app is designed. The next post will go into detail about how these tools were used and what lessons were learned.

The code can be found in the [easy-newspaper](https://github.com/IsaacVerm/easy-newspapers) repository. [Build results](https://travis-ci.org/IsaacVerm/easy-newspapers) can be found on Travis. If you want to play around with the app, best thing is to deploy it locally. Instructions how to do it can be found in the `README` of the GitHub repository.