---
layout: post
title: "Install tmate.io and share your terminal session"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - software_development
favorite: false
commentIssueId: 50
refimage: '/static/tmate.jpg'
---

This is an easy solution for sharing terminal sessions over ssh.
[Tmate.io](https://tmate.io) is great terminal sharing app,
you can think of it as Teamviewer for ssh.

![](/static/tmate.jpg)

To avoid compiling issues and dependencies, we will get the
static build directly from GitHub to `automagically` use it.

```bash
# Get files and install
wget https://github.com/tmate-io/tmate/releases/download/2.2.1/tmate-2.2.1-static-linux-amd64.tar.gz
tar -xvzf tmate-2.2.1-static-linux-amd64.tar.gz
sudo mv ./tmate-2.2.1-static-linux-amd64/tmate /usr/bin/
sudo chmod +x /usr/bin/tmate
rm -rf tmate-2.2.1-static-linux-amd64*

#Configure Tmate using ln2 as the default server
sudo tee -a ~/.tmate.conf > /dev/null <<'EOF'
set -g tmate-server-host "ln2.tmate.io"
EOF
```

And that is it, enjoy.

Use tmate and share the link...

### Running tmate as a daemon

You can run tmate detached, and retrieve
the SSH connection strings as follow:

```bash
tmate -S /tmp/tmate.sock new-session -d               # Launch tmate in a detached state
tmate -S /tmp/tmate.sock wait tmate-ready             # Blocks until the SSH connection is established
tmate -S /tmp/tmate.sock display -p '#{tmate_ssh}'    # Prints the SSH connection string
tmate -S /tmp/tmate.sock display -p '#{tmate_ssh_ro}' # Prints the read-only SSH connection string
tmate -S /tmp/tmate.sock display -p '#{tmate_web}'    # Prints the web connection string
tmate -S /tmp/tmate.sock display -p '#{tmate_web_ro}' # Prints the read-only web connection string
```

Note that it is important to specify a socket
(e.g. /tmp/dev.sock) as tmate uses a random
socket name by default.

You can think of tmate as a reverse ssh tunnel
accessible from anywhere.


Read more directly from [Tmate.io](https://tmate.io/).
