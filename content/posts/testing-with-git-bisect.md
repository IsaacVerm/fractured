---
title: "Testing with git bisect"
date: 2020-01-05T19:21:53+01:00
draft: false
---

## What's git bisect and why use it?

While browsing the git documentation, I noticed there are some debugging git commands I never heard about before. [git bisect](https://git-scm.com/docs/git-bisect-lk2009) in particular seemed interesting. With `git bisect` it's easy to track down when and where a bug was introduced. In this post I'll cover the same use cases as in the git documentation but with another example. This way I can be sure I thoroughly understood how to use `git bisect`. Having said that, I won't go into the inner workings of the bisect algorithm. 

The basic procedure of `git bisect` contains of the following steps:

- mark a commit as good
- mark a commit as bad
- go through one or more commits inbetween the good and bad commit
- for each commit decide if it's broken or not

By marking a commit as good and another one as bad, you define a range of commits in which the bug could have occured. The closer the good and bad commits can be, the less time it will take `git bisect` to pinpoint where things went wrong.

By doing a [binary search](https://en.wikipedia.org/wiki/Binary_search_algorithm) of commits, we end up with the culpable commit. Going through the commits can be done either automatically or manually. Conceptually it doesn't make a lot of difference since both approaches make a judgement on whether a commit broke something or not. Let's first have a look at a manual example.

## Manual bisecting

I added a branch `bisect-main` to the [git-examples repository](https://github.com/IsaacVerm/git-examples). The only file checked into this branch is `status` and the branch contains 10 commits. The first 5 commits are all fine since they either contain the words *good* or *still good*. However, from the sixth commit I started making mistakes and put either *bad* or *still bad* in `status`. To find where things went wrong I could manually check out the commits one by one and see where the text started changing. Next to being boring and long, this also only works in simple examples like the ones I used. Once you have to run more elaborate tests to verify if a commit introduced a bug or not, this approach quickly becomes very tedious. `git bisect` to the rescue.

Before getting started, make sure to have the `bisect-main` branch of the `git-examples` repository checked out. To start bisecting run `git bisect start`. If you run `git status` you will be notified you're bisecting. First thing to do now is mark a commit as good and a commit as bad. To get all the commits I run:

```
git log --oneline
```

For each commit you get its [hash](https://ericsink.com/vcbe/html/cryptographic_hashes.html) and message. 

```
5e19039 (HEAD -> bisect-main, origin/bisect-main) commit 10
f77e2fd commit 9
abef51d commit 8
58aa92a commit 7
c3c0223 commit 6
ecb3e7a commit 5
4517f34 commit 4
2dc4272 commit 3
4615bfa commit 2
7c6d6a0 commit 1
```

I know the first commit (`7c6d6a0`) is good so I run `git bisect good 7c6d6a0`. I'm looking at the last commit and see it's bad so I run `git bisect bad 5e19039`. A shortcut not to specify the hash is to run `git bisect bad` which automatically marks the current commit as bad.

After marking both a good and bad commit, `HEAD` gets detached to a commit somewhere in the middle. Where exactly is decided by the bisect algorithm. Since I have 10 commits it's reasonable for the first bisecting commit to be somewhere in the middle (since `git bisect` is based on using a binary search). Running `git log --oneline -1` shows it's commit 5 (out of 10).

The 5th commit is still a good one (we can see the word good in the `status` file) so we mark it as good by running `git bisect good`. We know the 5th commit is good and the last 10th commit is bad so the bug must have occured somewhere in between. `HEAD` will detach again to a commit in between commit 5 and commit 10. We repeat the same process until no more steps are proposed by `git bisect`. A message is displayed saying commit 6 is the first bad commit. This is correct!

Run `git bisect reset` to return to the tip of the branch.

## Bisecting with script

The manual procedure described above was effective but a bit cumbersome. For each commit proposed by the algorithm we had to manually mark it as either good or bad. Luckily there's also an option to run a script to do this testing for us. The only requirement for this script is it returns an exit code of 0 when the commit is good and anything else when something went wrong. A simple test in our case could be `grep "good" status`. This returns 0 when good is found in the `status` file and 1 it wasn't not found. We add this command to a script called `status-contains-good.sh`. It's possible to run it inline as well but I find that rather messy. Don't forget to make the script executable by running `chmod +x` on it.

The first part of the procedure is the same as for manual. We start `git bisect` and mark a commit as good and a commit as bad:

```
git bisect start;
git bisect good 7c6d6a0;
git bisect bad 5e19039;
```

However, instead of going through the commits one by one we now run `git bisect run ./status-contains-good.sh`. Just as before commit 6 is identified as the culpable.

If you run `git bisect log` you get to see exactly what happened:

```
git bisect start
# good: [7c6d6a0404899ca84012d2a2c80cdaaee6c43d42] commit 1
git bisect good 7c6d6a0404899ca84012d2a2c80cdaaee6c43d42
# bad: [5e19039f2e63a02347ec4931bfea987d93457062] commit 10
git bisect bad 5e19039f2e63a02347ec4931bfea987d93457062
# good: [ecb3e7af1c9f79ba1d3591f481eb7fc92b0ef2d0] commit 5
git bisect good ecb3e7af1c9f79ba1d3591f481eb7fc92b0ef2d0
# bad: [58aa92a1068ce75361967b895b894eff4712e17e] commit 7
git bisect bad 58aa92a1068ce75361967b895b894eff4712e17e
# bad: [c3c022349c7510803acae6adaac7787623688a15] commit 6
git bisect bad c3c022349c7510803acae6adaac7787623688a15
# first bad commit: [c3c022349c7510803acae6adaac7787623688a15] commit 6
```

As you can see the log file makes no difference between those commits marked good/bad by ourselves (the first and last ones) and those marked good/bad by the script (commit 5, 6 and 7).

Now you know where things went wrong but not which change caused the bug. If you use atomic commits (which is a best practice anyway) it becomes real easy to pinpoint the issue. Running `git show c3c0223` (hash of commit 6) you get all the changes introduced by commit 6:

```
commit c3c022349c7510803acae6adaac7787623688a15 (HEAD, refs/bisect/bad)
Author: IsaacVerm <isaacverm@gmail.com>
Date:   Mon Jan 6 11:07:09 2020 +0100

    commit 6

diff --git a/status b/status
index 476e93d..44d6628 100644
--- a/status
+++ b/status
@@ -1 +1 @@
-good
\ No newline at end of file
+bad
\ No newline at end of file
```

As you can see a line with good has been removed (`-`) and a line with bad has been added (`+`).

In real life tests are more complicated but in essence it's the same principle. Take for example the [easy newspaper](https://github.com/IsaacVerm/easy-newspapers) project I did some time ago. E2E tests using Cypress were added and you can leverage the power of these tests. If you ever notice something not right you can run the tests as a script with `npx cypress run`.