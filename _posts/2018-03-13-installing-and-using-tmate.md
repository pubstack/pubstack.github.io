---
layout: post
title: "Install tmate.io and share your terminal session"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - 'software development'
favorite: false
commentIssueId: 50
---

![](/static/tmate.jpg)

This is an easy solution for sharing terminal sessions over ssh.
[Tmate.io](https://tmate.io) is great terminal sharing app,
you can think of it as Teamviewer for ssh.

To avoid compiling issues and dependencies, we will get the
static build directly from GitHub to `automagically` use it.

```
wget https://github.com/tmate-io/tmate/releases/download/2.2.1/tmate-2.2.1-static-linux-amd64.tar.gz
tar -xvzf tmate-2.2.1-static-linux-amd64.tar.gz
sudo mv ./tmate-2.2.1-static-linux-amd64/tmate /usr/bin/
sudo chmod +x /usr/bin/tmate
```

And that is it, enjoy.

Open tmate and share the link...

