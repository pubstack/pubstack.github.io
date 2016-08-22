---
layout: post
title: "TripleO manual deployment of 'master' branch"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
commentIssueId: 2
---

This is a brief recipe about how to
manually install TripleO in a remote
32GB RAM box.

From the hypervisor run:

```bash
  #In this dev. env. /var is only 50GB, so I will create
  #a sym link to another location with more capacity.
  #It will take easily more tan 50GB deploying a 3+1 overcloud
  sudo mkdir -p /home/libvirt/
  sudo ln -sf /home/libvirt/ /var/lib/libvirt
  #Add default stack user
  sudo useradd stack
  echo "stack:stack" | chpasswd
  echo "stack ALL=(root) NOPASSWD:ALL" | sudo tee -a /etc/sudoers.d/stack
  sudo chmod 0440 /etc/sudoers.d/stack
  su - stack

  sudo yum -y install epel-release
  sudo yum -y install yum-plugin-priorities

  #Master repositories
  sudo curl -o /etc/yum.repos.d/delorean.repo http://buildlogs.centos.org/centos/7/cloud/x86_64/rdo-trunk-master-tripleo/delorean.repo
  sudo curl -o /etc/yum.repos.d/delorean-deps.repo http://trunk.rdoproject.org/centos7/delorean-deps.repo

  #Configure the undercloud deployment
  export NODE_DIST=centos7
  export NODE_CPU=1
  export NODE_MEM=7550
  export NODE_COUNT=3
  export UNDERCLOUD_NODE_CPU=1
  export UNDERCLOUD_NODE_MEM=9000
  export FS_TYPE=ext4

  sudo yum install -y instack-undercloud
  instack-virt-setup
```

In the hypervisor run the following command to log-in in
the undercloud:

```bash
  ssh root@`sudo virsh domifaddr instack | grep $(tripleo get-vm-mac instack) | awk '{print $4}' | sed 's/\/.*$//'`
```

From the undercloud we will install all the
packages:

```bash
  #Add a 4GB swap file to the Undercloud
  sudo dd if=/dev/zero of=/swapfile bs=1024 count=4194304
  sudo mkswap /swapfile
  #Turn ON the swap file
  sudo chmod 600 /swapfile
  sudo swapon /swapfile
  #Enable it on start
  sudo echo "/swapfile          swap            swap    defaults        0 0" >> /etc/fstab

  #Login as the stack user
  su - stack
  sudo yum -y install yum-plugin-priorities
  export USE_DELOREAN_TRUNK=1
  export DELOREAN_TRUNK_REPO="http://buildlogs.centos.org/centos/7/cloud/x86_64/rdo-trunk-master-tripleo/"
  export DELOREAN_REPO_FILE="delorean.repo"
  export FS_TYPE=ext4

  git clone https://github.com/openstack/tripleo-heat-templates
  git clone https://github.com/openstack-infra/tripleo-ci.git
  
  ./tripleo-ci/scripts/tripleo.sh --all
  #The last command will execute:
  #repo_setup
  #undercloud
  #overcloud_images
  #register_nodes
  #introspect_nodes
  #overcloud_deploy
```

Once the undercloud it is fully installed, deploy an overcloud
(The last command should have created an overcloud, this is
needed if you need to deploy another one).

```bash
cd
openstack overcloud deploy \
--libvirt-type qemu \
--ntp-server pool.ntp.org \
--templates /home/stack/tripleo-heat-templates \
-e /home/stack/tripleo-heat-templates/overcloud-resource-registry-puppet.yaml \
-e /home/stack/tripleo-heat-templates/environments/puppet-pacemaker.yaml
#Also can be added:
#--control-scale 3 \
#--compute-scale 3 \
#--ceph-storage-scale 1 -e /home/stack/tripleo-heat-templates/environments/storage-environment.yaml
```

This will hopefully deploy the TripleO overcloud, if not,
refer to the [troubleshooting](http://tripleo.org/troubleshooting/troubleshooting.html) section in the official
site.

```bash
#Configure a DNS for the OC subnet, do this before deploying the Overcloud
neutron subnet-update `neutron subnet-list -f value | awk '{print $1}'` --dns-nameserver 192.168.122.1
```
