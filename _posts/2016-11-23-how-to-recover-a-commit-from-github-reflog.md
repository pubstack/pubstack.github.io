---
layout: post
title: "How to recover a commit from GitHub's Reflog"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - software_development
favorite: true
commentIssueId: 22
refimage: '/static/crashed-those-commits.png'
---

Writing
[this](http://www.pubstack.com/blog/2016/11/21/openstack-summit-2016-bcn.html) blog post,
suddenly and without knowing I ended up by squashing/removing the commit
holding those changes.

![](/static/crashed-those-commits.png)

The first thing that came to my mind was to locally check the Reflog to
restore the commit, but sadly I was removed the repo from my laptop and the
Reflog didn't exist anymore. Then I went into panic as I didn't wanted to
lose those hours that took me to write the post.
Then, I said... Does GitHub have Reflog? The sweet answer, yes..

So lets learn how to recover a commit from GitHub Reflog.

Relevant strings to fill:

- \<user\>: The user holding the git repo.
- \<repo\>: The repository name.
- \<recover-branch-name\>: The remote branch that you will create.
- \<sha-goes-here\>: The commit sha to be restored.

Let's curl GitHub to get all commits in the Reflog:

```
$ curl https://api.github.com/repos/<user>/<repo>/events
```

Now let's create a remote branch with your commit.

Choose your commit sha and run:

```
$ curl -i -H "Accept: application/json" -H "Content-Type: application/json" -X POST -d '{"ref":"refs/heads/<recover-branch-name>", "sha":"<sha-goes-here>"}' https://api.github.com/repos/<user>/<repo>/git/refs
```

You should have created a branch called <recover-branch-name>
in your GitHub repo. Now you can safely cherry-pick it to your
master branch or fetch locally those changes and do
whatever you want with them.

So yeah, I have to admit that I <s>squashed</s>crashed those commits in some how...

I hope that last tips are useful if you are in a trouble like me
trying to recover some lost commits from GitHub.
