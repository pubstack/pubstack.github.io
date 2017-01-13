---
layout: post
title: "Installing TripleO UI"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
commentIssueId: 27
---

This is a brief recipe to install TripleO UI
in the Undercloud.

Lets install the TripleO UI and
all the prerequisites.

```bash
  cd
  yum install -y tmux
  git clone https://github.com/openstack/tripleo-ui.git
  sudo yum install -y nodejs npm
  cd tripleo-ui
  npm install
```

Now, update all the TripleO UI config files

```bash
  cd
  cp ~/tripleo-ui/dist/tripleo_ui_config.js.sample ~/tripleo-ui/dist/tripleo_ui_config.js
  echo "Changing the default IP"
  export ENDPOINT_ADDR=$(cat stackrc | grep OS_AUTH_URL= | awk -F':' '{print $2}'| tr -d /)
  sed -i "s/[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}/$ENDPOINT_ADDR/g" ~/tripleo-ui/dist/tripleo_ui_config.js
  echo "Removing comments"
  sed -i '/^  \/\/ \".*\"\:/s/^  \/\///' ~/tripleo-ui/dist/tripleo_ui_config.js
  echo "Changing listening port for the dev server, 3000 already used"
  sed -i '/port: 3000/s/3000/12121/' ~/tripleo-ui/webpack.config.js
```

In the following step we will use tmux to persist the service running
for debugging purposes.

```bash
  cd
  tmux new -s tripleo-ui
  cd ~/tripleo-ui/
  npm start
```

At this stage you should have up and running your node server.
Now lets create a tunnel to connect to the UI.

```bash
  #Forward incoming 38080 traffic to local 38080 in the hypervisor
  #labserver must be a reachable host
  ssh -L 38080:localhost:38080 root@labserver
  #Log-in as the stack user and get the undercloud IP
  su - stack
  undercloudIp=`sudo virsh domifaddr instack | grep $(tripleo get-vm-mac instack) | awk '{print $4}' | sed 's/\/.*$//'`
  #Forward incoming 38080 traffic to local 12121 in the undercloud (Where we have the TripleO UI running)
  ssh -L 38080:localhost:12121 root@$undercloudIp
  su - stack
  source stackrc
  echo "Copy the password to use it later..."
  echo $OS_PASSWORD
```

Save the echoed password and use it together with the admin user to log in the TripleO UI:  http://localhost:38080/

Happy TripleOing!

<div style="font-size:10px">
  <blockquote>
    <p><strong>Updated 2017/01/13:</strong> First version.</p>
  </blockquote>
</div>
