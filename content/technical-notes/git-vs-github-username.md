---
title: "git and GitHub username are different things"
date: 2020-01-06T14:41:56+01:00
draft: false
---

> Code for this post can be found in the [config-username](https://github.com/IsaacVerm/git-examples/tree/config-username) branch of the git-examples repository.

[Your username on GitHub does not have to match your git username](https://help.github.com/en/github/using-git/setting-your-username-in-git). Let's say I change my name to "a" and commit a file called whoami. This file is pushed to GitHub.

```
git config --global user.name "a";
echo a >> whoami;
git commit -m "whoami a"
git push;
```

I can see my name has changed:

```
git config user.name
```

Now let's change my name to "b":

```
git config --global user.name "b"
echo b >> whoami;
git commit -m "whoami b"
git push;
```

With the following command I can see which users were responsible for each commit ([git log formatting](https://git-scm.com/docs/git-log) is quite powerful by the way).

```
git log --pretty="format:%s - %an" -2
```

Which gives:

```
whoami b - b
whoami a - a
```

The first column is the commit message (the `%s` subject part), the part after the dash is the author of the commit (`%an`). As you can see the commits are linked to 2 different authors because we changed our name locally inbetween commits. On GitHub however you won't see this because there only the GitHub username is displayed:

![github username change](/github-username-change.png)

