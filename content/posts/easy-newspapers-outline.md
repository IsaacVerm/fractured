---
title: "Web app for easy access to newspapers"
date: 2019-12-02T14:22:24+01:00
draft: false
---

## Use case

For elderly people it can be hard to get access to newpaper articles since the new GDPR regulations.

![cookie prompt](/cookie-prompt.png)

There's many ways things can go south from here by tapping buttons you don't want to select. By making a small web app wrapping RSS feeds from newspapers we should be able circumvent the complexity. An example of a newspaper providing a RSS feed is [De Standaard](https://www.standaard.be/rssfeeds).

At a minimum you should be able to:

- see a summary of the RSS feed of a newspaper
- see articles saved
- send mail if you find an interesting article

## Technology

The web app is built in [Django](https://www.djangoproject.com/) and tested E2E with [Cypress](https://www.cypress.io/). Deployment is done on [Heroku](https://www.heroku.com/), continuous integration is handled by [Travis](https://travis-ci.org/).

I'd like to do some [test-driven development](https://en.wikipedia.org/wiki/Test-driven_development), a way of programming in which tests are written before the actual code.

### Why Django?

From the [Django main page](https://www.djangoproject.com/):

> Django is a high-level Python Web framework that encourages rapid development and clean, pragmatic design.

This is exactly what I'm looking for. The idea behind this post is to focus on the E2E testing part in Cypress. The web app serves just as a practical example. I have used Django previously and know how easy it is to get something up and running fast. Added benefit is it's Python and it's been a while since I did something in Python.

Also Django has a lot of out of the box tools to help with common tasks. I'm at least going to need to have the following:

- mail
- authentication
- database

Django has an [object-relational mapper](https://docs.djangoproject.com/en/3.0/topics/db/models/) which means you can specify what your database should be like in Python code. The model is the single source of truth.

Django can set up an [authentication system](https://docs.djangoproject.com/en/2.2/topics/auth/) with some minimal added configuration.

Django also provides [mail functionality](https://docs.djangoproject.com/en/2.2/topics/email/).

If you browse through these articles, you see everything is documented and it's documented well. It's always a boon not to have to figure the basics out over and over again.

[Ruby on Rails](https://rubyonrails.org/) seems like a good alternative to Django using the same Model-View-Controller framework. However, I don't want to learn Ruby for now and just focus on Python.

### Why Cypress?

I've used Cypress at work and have good experience with it. I opted for only E2E tests since I don't want to get into any unit testing or API testing for this project. Since I still want to have the reassurance things work I focus on the E2E part.

There are numerous advantages to using Cypress but off the top my head what I liked most so far is:

- the debugger where you can see the state of the DOM at any time
- the integration with DevTools (especially handy if combined with the debugger)
- can automatically take screenshots (and video but never used that feature)

### Why Heroku?

I used Heroku before. There are a few [reasons not to use Heroku](https://medium.com/@brenda.clark/heroku-alternatives-top-5-picks-9095cef91d91) but non of those apply to this project at the moment. Django is [supported by Heroku](https://devcenter.heroku.com/articles/django-app-configuration), I don't need to do anything fancy and I'm still in the free tier. Since Heroku is so widely used it's easy to find quick fixes if running into any issues. Google has an alternative with [Google Cloud](https://cloud.google.com/) but I didn't really see any added benefit over using Heroku.

### Why Travis?

Travis I also used before to continuously test [R code](https://travis-ci.org/IsaacVerm/marriage) and [Flask apps](https://github.com/IsaacVerm/postcards-tests/blob/master/.travis.yml). The process was simple and you just have to make sure you have a `.travis.yml` configuratoin file and follow the Travis conventions.

The main alternative for Travis seems to be [CircleCI](https://circleci.com/). Travis seems to have [a bit more features](https://hackernoon.com/continuous-integration-circleci-vs-travis-ci-vs-jenkins-41a1c2bd95f5) than CircleCI but in essence they work pretty much the same way.

## Approach

The idea is to first set up a [very small example](https://homepage.cs.uri.edu/~thenry/resources/unix_art/ch01s07.html) combining all the technologies I plan to use.

So the minimum requirements are:

- the web app has the summary for a single newspaper
- this summary is covered by an E2E test
- deployment of the app is automated on each change
- continuous integration

## Web app

What's mentioned already in the Django getting started guide and is clear there is not rehashed here.

### structure

The following pages exist:

- articles feed (articles_feed)
  - title
  - publication date
- articles sent by mail (articles_sent_by_mail)
  - title
  - publication date
  - sent to who
- saved articles (articles_saved)
  - title
  - publication date
- article detail (article_detail)
  - title
  - description
  - can save article
  - can send article by mail

The article detail page can be accessed both from:

- the articles feed
- articles sent by mail
- saved articles

The articles saved page can be accessed both from:

- after saving an article
- main page

From the detail page you can always go to each of those.

You shouldn't be able to save an article already saved. You shouldn't be able to send an article by mail which has already been sent.

Implementing [a form](https://developer.mozilla.org/en-US/docs/Learn/HTML/Forms/Your_first_HTML_form) is a [bit more involved](https://docs.djangoproject.com/en/3.0/intro/tutorial04/).

### correct field type

It's important to choose the correct field type. E.g. [CharField](https://docs.djangoproject.com/en/3.0/ref/models/fields/#charfield) is used for short texts, [TextField](https://docs.djangoproject.com/en/3.0/ref/models/fields/#django.db.models.TextField) is used for longer texts.

### migrations

Migrating actually consists of 2 commands:

- `makemigrations`
- `migrate`

`makemigrations` means you want to turn the changes to your model into a migration. `migrate` reflects those migrations in the database.

For safety reasons you can run 2 commands:

- `sqlmigrate`
- `check`

`sqlmigrate` means you want to see what changes will be made in the database. This doesn't execute anything. `check` works like `devtools::check()` in R.

### meta database API

Next to the [database API](https://docs.djangoproject.com/en/3.0/topics/db/queries/) itself, there's also a [model meta API](https://docs.djangoproject.com/en/3.0/ref/models/meta/). You can use meta api to get for example all the fields defined in a model (with their type, ...). Within the Django shell:

```
Article._meta.get_fields()
```

### admin

Getting a fullblown admin interface for free is great. Only thing to do is register the models you want to be able to edit in `admin.py`.

Form automatically generated:

![django form autogenerated](/django-form-auto-generated.png)

### template namespacing

Template namespacing is weird because you end up with `/articles/templates/articles` but it's necessary because of the [way Django loads templates](https://docs.djangoproject.com/en/3.0/intro/tutorial03/).

### filter on model methods

[You can't filter on model methods](https://stackoverflow.com/questions/31658793/django-filter-query-on-with-property-fields-automatically-calculated). The correct way to filter is to use list comprehensions:

- first get all the results in the queryset
- filter manually with conditional statement like if

There's a solution as well where you add the result of the method as a field in the model. But this way the model becomes bloated. Queries are sent to the database so they can only filter on fields which exist in the database.

### code in templates

[There's no Python code in templates](https://docs.djangoproject.com/en/3.0/ref/templates/language/) although some stuff may have the same name (e.g. `for`).

### while implementing

Put calls to external API (rss feeds in our case) [in views function](https://simpleisbetterthancomplex.com/tutorial/2018/02/03/how-to-use-restful-apis-with-django.html).

`feedparser` doesn't work anymore because of [SSL certification issue](https://stackoverflow.com/questions/28282797/feedparser-parse-ssl-certificate-verify-failed).

Must use `{% csrf_token %}` for [CSRF errors](https://docs.djangoproject.com/en/3.0/intro/tutorial04/).

Use `reverse` not to hardcode urls in templates. Maybe not so important since the app is still small. Make issue in Github.

Proper url management is key.

Use [repath](https://docs.djangoproject.com/en/3.0/topics/http/urls/) to make distinction between `<article_path>` and save.

Issue: reusable functions between views. E.g. clean article_path involved removing `de standaard blabla`.

Working with forms can be made easier by using the [Form](https://docs.djangoproject.com/en/3.0/topics/forms/#the-view) class.

Hack: hidden `input` element prefilled with description data. Probably not the proper way to pass data between contexts and views.

Multiple forms so at least one form is empty results in [multivalue dict key errors](https://stackoverflow.com/questions/5895588/django-multivaluedictkeyerror-error-how-do-i-deal-with-it).

### sending mails

Sending mails itself is [well documented and easy](https://docs.djangoproject.com/en/3.0/topics/email/).

To send mails you have to provide your password. Checking this password in to GitHub is naturally not a good idea since it's there for everyone to see. You can set up [environment variables](https://stackoverflow.com/questions/55640495/django-send-mail-security-password) which are injected at runtime. Environment variables in Python are set [this way](https://djangostars.com/blog/configuring-django-settings-best-practices/).

### css

button as hyperlink is [hard](https://www.w3docs.com/snippets/html/how-to-create-an-html-button-that-acts-like-a-link.html). Played around with `input` from form. Settled for a button in the end.

## Cypress

Now we've got the mimimum of an app set up, it's time to set up the testing. I chose to have the tests in the same repository as the web app. Figured I can always split it off in a separate repo later on if the need might arise. Also opens possibility of [watching app file changes](https://github.com/bahmutov/cypress-watch-and-reload).

Important to test [font size](https://kyleschaeffer.com/css-font-size-em-vs-px-vs-pt-vs-percent) for elderly.

Data has to be [seeded](https://docs.djangoproject.com/en/2.2/howto/initial-data/).

## Travis

### notes

We can only test against a running server. At first I tried to have 2 separate jobs:

- a python job running a local version of the web server
- node job with tests against this local web server

Somehow this worked. I have no idea why the current setup works. Python process serving the Django server should never finish so the Cypress tests shouldn't run in the job just afterwards. But after some time the tests do start to run.

Sadly discovered the node job has access to the local web server. So I moved to a different approach:

- first deploy to Heroku
- then run tests on Travis against this deployed Heroku version

So there's no longer any need for build matrices, ...

That's why the latest changes are first deployed to Heroku as explained [here](https://docs.travis-ci.com/user/deployment/heroku/).

Travis can only handle [single commands or scripts](https://docs.travis-ci.com/user/deployment/script/).

You can run multiple languages [in the same Travis config](https://stackoverflow.com/questions/27644586/how-to-set-up-travis-ci-with-multiple-languages).

Difference between [build matrix](https://docs.travis-ci.com/user/build-matrix/) and [build stages](https://docs.travis-ci.com/user/build-stages/) is the order. Jobs in the build matrix run in parallel while those in build stages run sequentially.

Travis still fails, but because it doesn't share the web server between jobs. Didn't find a easy way to make this possible. Maybe best way is to move on to Heroku deployment, deploy the web app to a test environment and just then run the tests.

### pays off fast

Since on every push to Github the tests ran, you immediately get notified if something goes wrong. Even if you think nothing could have gone wrong. Good catch was this commit where I [forgot to add](https://github.com/IsaacVerm/easy-newspapers/commit/7274eb501312c2ba80b6cee2608b5f25377438c5) one of the installed packages to `requirements.txt`. The app broke down and I could immediately handle the issue. If I would have continued developing I wouldn't have noticed anything (since I do have `atoma` locally!). Untangling the mess surely would have taken some time.

## Heroku

Interesting [explanation by Mozilla](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Deployment) how to deploy Heroku.

Heroku provides a library [helping with deploying Django apps](https://github.com/heroku/django-heroku).

You should install `gunicorn` if you wan't to deploy (but not needed when deployed).

We can't use the default SQLite database on Heroku because it is file-based, and it would be deleted from the ephemeral file system every time the application restarts (typically once a day, and every time the application or its configuration variables are changed).

The Heroku mechanism for handling this situation is to use a database add-on and configure the web application using information from an environment configuration variable, set by the add-on.

[Github] Django template.
