---
title: "change editor used by git"
date: 2020-01-06T15:38:32+01:00
draft: false
---

By default the Mac installation of git uses vim as editor. If you're not familiar with vim this can be quite hard. I'm used to Visual Code and it's easy to configure Visual code as the default:

```
git config --global core.editor "code"
```

You can see for yourself the configuration has been updated. Either by running the `git config` command with the `--list` option:

```
git config --list | grep core.editor
```

Or by verifying the config file itself. You can find the path of the config file by adding the `--show-origin` option.

```
git config --list --show-origin | grep core.editor;
cat GIT_CONFIG_FILE_PATH | grep editor
```

If you know run a command which uses the editor (like for example `git commit`) Visual Code will be opened.

![git commit visual code](/git-commit-visual-code.png)