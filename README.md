# README

## Install Hugo

On [Mac OS](https://gohugo.io/getting-started/quick-start/):

```
brew install hugo
```

## Create blog

```
hugo new site blog
```

## Create post

```
hugo new posts/blablabla.md
```

Post will be added to `/content/posts` folder.

## Start server

If you just use `new posts` the post created will have draft status set to true. This means the post will not be published. We want to see what the post looks like so we have to override this default behaviour. Hence the `-D` argument for the server.

```
hugo server -D
```

## Deployment

Deployed on [GitHub Pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/).

### Steps

First make sure the repository containing the blog has been [set up](https://help.github.com/en/articles/configuring-a-publishing-source-for-github-pages) as the one you use for GitHub Pages.

Build:

```
hugo
```

Add commit with build in `/public` included:

```
git add -A
git commit -m "name changes"
```

Push commit to master branch:

```
git push
```

Tip: if you have previously deployed with another static site generator (I used Jekyll before) you have to [unpublish](https://help.github.com/en/articles/unpublishing-a-project-pages-site) the previous one first. This can be changed in the settings of the repository on GitHub.

## Static content

[Static content](https://gohugo.io/content-management/static-files/) like images are put in the `/static` folder.

Important remark, images can be added like this in a post:

```
![Example image](/neocities_dashboard.png)
```

Locally you can get away with not using an alt label like `Example image` but you need it in Hugo.

## Theme

Uses [ezhil](https://github.com/vividvilla/ezhil) theme.
