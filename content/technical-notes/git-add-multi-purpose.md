---
title: "git add is a multipurpose command"
date: 2020-01-06T16:33:12+01:00
draft: false
---

`git add` has [multiple uses](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository). To follow along create an empty repository:

```
mkdir repo;
cd repo;
git init;
```

## Adds untracked files to the index

First create a new file with `touch test`. If we run `git status -s` now, you can see the file is untracked because `?? test` is displayed (a full explanation of each abbreviation in short format can be found [here](https://git-scm.com/docs/git-status#_short_format)). 

Now we can update the index:

```
git add test
```

The output of `git status -s` changes to `A  test`. A for added because the file is added to the index.

## Adds modified files to the index

Suppose we make a commit with test included:

```
git commit -m "add test"
```

We make a change to the test file and add the file again:

```
echo a >> test;
git add test
```

Instead of showing `?? test` as it would for untracked files, it now shows `M test`.

## Marks merge conflicts as resolved

A merge conflict can occur in several situations but one of the most common ones is if [the same file is changed in 2 different commits](https://www.atlassian.com/git/tutorials/using-branches/merge-conflicts). The easiest way to simulate this is as follows:

- we create a file `test` in the main branch and commit it
- we create a new branch based on this main branch in which we change `test`
- we update `test` again in the main branch and commit it again

You have to make sure changes occur on the same line of the file. If not there won't be a merge conflict since git can automatically resolve situations in which conflicts occur on different line numbers.

First set up the main branch:

```
echo a > test;
git add test;
git commit -m "add a"
```

Now we create a conflict branch based on this main branch:

```
echo b > test;
git add test;
git commit -m "add b"
```

Almost there, just modify the file once more in the main branch. If not there won't be any conflict. The main branch in the [git-examples](https://github.com/IsaacVerm/git-examples) repository is called `add-multi-purpose` but you can call it whatever you want.

```
git checkout add-multi-purpose;
echo c > test;
git add test;
git commit -m "add c"
```

Now we can trigger the conflict:

```
git merge conflict-branch
```

`git` automatically modifies the `test` file so it's easier to see where the conflicts arose:

```
<<<<<<< HEAD
c
=======
b
>>>>>>> conflict-branch
```

You have to pick an option by removing the arrows and equal signs not needed. Let's say we only want to keep our latest change to c. Now we you run `git add test` a message will appear saying all conflicts have been fixed (there was only one). Pay attention, the merge is only complete after running `git commit`.  If you run `git log --oneline -1` you'll see a merge commit has been created.