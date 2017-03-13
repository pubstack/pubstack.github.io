---
layout: post
title: "Installing TripleO Quickstart"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
favorite: true
commentIssueId: 29
---

This is a brief recipe about how to
manually install TripleO Quickstart in a remote
32GB RAM box and not dying trying it.

Now `instack-virt-setup` is deprecated :( :( :(
so the manual process needs to evolve and use OOOQ (TripleO Quickstart).

This post is a brief recipe about how to provision the Hypervisor node
and deploy an end-to-end development environment
based on TripleO-Quickstart.

From the hypervisor run:

```bash
  # In this dev. env. /var is only 50GB, so I will create
  # a sym link to another location with more capacity.
  # It will take easily more tan 50GB deploying a 3+1 overcloud
  sudo mkdir -p /home/libvirt/
  sudo ln -sf /home/libvirt/ /var/lib/libvirt
  # Add default toor user
  sudo useradd toor
  echo "toor:toor" | chpasswd
  echo "toor ALL=(root) NOPASSWD:ALL" | sudo tee -a /etc/sudoers.d/toor
  sudo chmod 0440 /etc/sudoers.d/toor
  su - toor
  whoami

  sudo yum groupinstall "Virtualization Host" -y
  sudo yum install git -y

  cd
  mkdir .ssh
  ssh-keygen -t rsa -N "" -f .ssh/id_rsa
  cat .ssh/id_rsa.pub >> .ssh/authorized_keys
  sudo bash -c "cat .ssh/id_rsa.pub >> /root/.ssh/authorized_keys"
  sudo bash -c "echo '127.0.0.1 127.0.0.2' >> /etc/hosts"
  export VIRTHOST=127.0.0.2

  ssh root@$VIRTHOST uname -a

  # Use: --release [newton|ocata|master] as described in the next commands.

  git clone https://github.com/openstack/tripleo-quickstart
  chmod u+x ./tripleo-quickstart/quickstart.sh
  bash ./tripleo-quickstart/quickstart.sh --install-deps --release master


  printf "\n\nSee:\n./tripleo-quickstart/quickstart.sh --help for a full list of options\n\n"

  # 3 controller nodes and 3 compute nodes
  # Controller mem 8192 Compute mem 8192 Undecloud mem 12288
  # Will take around 60GB
  # bash ./tripleo-quickstart/quickstart.sh --release master --tags all --config /home/toor/tripleo-quickstart/config/general_config/ha_big.yml $VIRTHOST

  # 3 controller nodes and 1 compute node
  # Controller mem 6144 Compute mem 6144 Undecloud mem 8192
  # Will take around 32GB
    bash ./tripleo-quickstart/quickstart.sh --release master --tags all --config /home/toor/tripleo-quickstart/config/general_config/ha.yml $VIRTHOST


```

In the hypervisor run the following command to log-in in
the undercloud:

```bash
  ssh -F /home/toor/.quickstart/ssh.config.ansible undercloud
```

At this point you should have your development environment deployed correctly.

For further environment configurations
check the content of
`/home/toor/tripleo-quickstart/config/general_config/`

Note: `virsh list --all` will work only as the stack user.

Note: Each time you run the quickstart.sh command from the hypervisor
the UC and OC will be nuked, you will see tasks like 'PLAY [Tear down undercloud and overcloud vms] **'.

Note: If you delete the Overcloud i.e. using `heat stack-delete overcloud` you can re-deploy what you
had by running the dynamically generated overcloud-deploy.sh script in the stack home folder from the UC.

Note: There are several options for TripleO Quickstart besides the basic 
virthost deployment, check them here: `https://docs.openstack.org/developer/tripleo-quickstart/working-with-extras.html`

<div style="font-size:10px">
  <blockquote>
    <p><strong>Updated 2017/02/27:</strong> Working fine.</p>
    <p><strong>Updated 2017/02/23:</strong> Seems to work.</p>
    <p><strong>Updated 2017/02/23:</strong> instack-virt-setup is deprecatred :( moving to tripleo-quickstart.</p>
  </blockquote>
</div>
