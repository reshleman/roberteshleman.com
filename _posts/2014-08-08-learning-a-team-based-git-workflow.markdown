---
layout: post
title: "Learning a Team-Based Git Workflow"
date: 2014-08-08 14:37:09 -0400
comments: true
redirect_from: /blog/2014/08/08/learning-a-team-based-git-workflow/
categories: ["Metis", "Git", "tech", "GitHub", "agile", "workflow"]
---


*This summer, I'm learning Ruby on Rails at [Metis], a 12-week class taught by some great folks from [thoughtbot]. This post is part of a series sharing my experience and some of the things I'm learning.*

[Metis]: http://www.thisismetis.com
[thoughtbot]: http://www.thoughtbot.com

At Metis this week, we've transitioned from learning in a classroom environment to collaborating with a team on our first projects. Among the challenges we've encountered, mastering a [Git] workflow stands out as one of the most useful — but also intimidating — skills to learn.

[Git]: http://en.wikipedia.org/wiki/Git_%28software%29

<!-- More -->

## Working in Feature Branches

In the past, when I've used Git for personal projects without collaborators, I've just written, committed, and pushed code on the `master` branch without any problems. Working with a team, though, `master` must be reserved for the *master* version of the code — the most up-to-date, final, and (hopefully) working version.

Changes that any team member is working on should be made in a separate Git feature branch, which protects `master` from unintentional changes. The convention we've used is to name a branch with your initials and a short description of the feature you're writing, like this: `re-sign-in-users`. This helps other developers identify who the owner of the branch is, and keeps your branches small by ensuring your work is directly related to that feature.

To make a new branch, we first checkout `master`:

```
git checkout master
```

It's also a good practice to ensure your `master` branch is up to date with the remote repository by pulling down any changes to master that have been merged in by another developer:

```
git pull origin master
```

Then, we can checkout our new feature branch:

```
git checkout -b re-sign-in-users
```

## Code Reviews on GitHub

Now that we're on a new branch, we can add, change, and delete code without worrying about modifying `master`. Let's say we work on our feature, add a few commits, and we think our code is ready to ship. Before we merge our feature into `master`, we want our teammates to review our code, and [GitHub] gives us a great tool for that: [pull requests].

First, let's push our code to GitHub. From our local branch, we run:

```
git push origin re-sign-in-users
```

This creates the branch on GitHub if it doesn't exist, and pushes the local branch up to it. GitHub will usually recognize your new branch immediately (AJAX magic!), and you'll see a green "Compare & pull request" button. GitHub gives you space to add a title and any notes for your team to understand the changes you've made. I also like to review my own changes one last time to ensure everything looks correct before clicking "Create pull request."

Once the PR has been created, our team can review the code and make comments directly in the PR. I've found this guide on [how to read a pull request] (written by [Metis] instructor [Gabe Berke-Williams]) to be incredibly helpful when reading my teammates' PRs.

[GitHub]: https://github.com
[pull requests]: https://help.github.com/articles/using-pull-requests
[how to read a pull request]: https://gist.github.com/gabebw/39adacfb03c7308644de
[Metis]: http://www.thisismetis.com
[Gabe Berke-Williams]: http://gabebw.com/

## Rebase and Merge

After your teammates have approved the code, it's time to merge it. There are several approaches to incorporating code from a feature branch into `master`; we've been using a workflow that involves **rebasing**. 

Until I understood what it really meant, "rebase" was a frightening word to me. It shouldn't be. In our case, [rebasing] involves making our changes be *based* off of the newest commit on `master`, instead of an older commit from when the feature branch was initially created. This allow us to incorporate any changes that our teammates have made and merged into `master` since we started writing our feature.

An additional benefit to rebasing is that it allows us to *squash* our commits, or make multiple discrete commits become one cohesive commit. This keeps our commit history on `master` clean, because each feature branch becomes only a commit or two, regardless of how many individual commits you made while developing the feature.

There are a few steps here, but the process becomes much easier with practice.

We want to start by making sure our local `master` branch has the most up-to-date changes from the remote repo:

```
git checkout master
git pull origin master
```

Then, we can rebase our branch on master:

```
git checkout re-sign-in-users
git rebase -i master
```

The `-i` flag on `rebase` triggers an `interactive` rebase, opening a text editor with a list of the commits from the branch. This is where we have the option of squashing our commits. I won't go into too much detail here, but GitHub has a great [rebase tutorial] about the interactive rebase process.

There's also a possibility that we'll have to resolve merge conflicts if our branch has changes that conflict with `master`. [Resolving merge conflicts] during a rebase is the same as during a normal merge, and again GitHub has a great guide explaining this process.

Once our rebase is done, we can merge our changes into `master` and push them up to GitHub. Because we've done the hard work in the rebase, the merge itself is easy:

```
git checkout master
git merge re-sign-in-users
git push origin master
```

[rebasing]: https://www.atlassian.com/git/tutorial/rewriting-git-history#!rebase
[rebase tutorial]: https://help.github.com/articles/using-git-rebase
[Resolving merge conflicts]: https://help.github.com/articles/resolving-a-merge-conflict-from-the-command-line

## Clean Up after Yourself

After we've merged our feature branch into `master`, we should clean up after ourselves. It's a good practice to remove unused branches from the repository — they're just clutter and it they can make it unclear which branches are actively being used. Fortunately, this step is pretty easy.

From the command line, we delete our local feature branch:

```
git branch -d re-sign-in-users
```

We can also delete our remote GitHub branch using the command line:

```
git push origin :re-sign-in-users
```

Note the colon before the feature branch name. This syntax allows us to specify which local branch to push from, and which remote branch to push to. In this case, we're pushing a local branch of nothing (before the colon) to the remote branch `re-feature-branch`. Pushing "nothing" tells Git to delete the remote branch.

GitHub will kindly close the pull request for you when the remote branch is deleted. Neat!

## Resources

Here's a quick summary of a few of the most helpful resources I've found on these topics:

* [Overview of pull requests]
* [Reading a pull request]
* [Overview of rebasing]
* [GitHub rebasing tutorial]
* [GitHub article on resolving merge conflicts]

[Overview of pull requests]: https://help.github.com/articles/using-pull-requests
[Reading a pull request]: https://gist.github.com/gabebw/39adacfb03c7308644de
[Overview of rebasing]: https://www.atlassian.com/git/tutorial/rewriting-git-history#!rebase
[GitHub rebasing tutorial]: https://help.github.com/articles/using-git-rebase
[GitHub article on resolving merge conflicts]: https://help.github.com/articles/resolving-a-merge-conflict-from-the-command-line
