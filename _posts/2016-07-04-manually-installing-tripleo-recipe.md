---
layout: post
title: "TripleO manual deployment - DEPRECATED"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
  - cloud
commentIssueId: 2
refimage: '/static/tripleo_banner.png'
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

  export TRIPLEO_ROOT=/home/stack
  export TRIPLEO_RELEASE=rdo-trunk-master-tripleo
  #export TRIPLEO_RELEASE=rdo-trunk-newton-tested
  export TRIPLEO_RELEASE_DEPS=centos7
  #export TRIPLEO_RELEASE_DEPS=centos7-newton

  #Repository configured pointing to above release!
  sudo curl -o /etc/yum.repos.d/delorean.repo https://buildlogs.centos.org/centos/7/cloud/x86_64/$TRIPLEO_RELEASE/delorean.repo
  sudo curl -o /etc/yum.repos.d/delorean-deps.repo https://trunk.rdoproject.org/$TRIPLEO_RELEASE_DEPS/delorean-deps.repo

  #Configure the undercloud deployment
  export NODE_DIST=centos7
  export NODE_CPU=4
  export NODE_MEM=9000
  export NODE_COUNT=6
  export UNDERCLOUD_NODE_CPU=4
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

  export TRIPLEO_ROOT=/home/stack
  sudo yum -y install yum-plugin-priorities

  export TRIPLEO_RELEASE=rdo-trunk-master-tripleo
  #export TRIPLEO_RELEASE=rdo-trunk-newton-tested
  export TRIPLEO_RELEASE_BRANCH=master
  #export TRIPLEO_RELEASE_BRANCH=stable/newton

  export USE_DELOREAN_TRUNK=1
  export DELOREAN_TRUNK_REPO="https://buildlogs.centos.org/centos/7/cloud/x86_64/$TRIPLEO_RELEASE/"
  export DELOREAN_REPO_FILE="delorean.repo"
  export FS_TYPE=ext4

  git clone -b $TRIPLEO_RELEASE_BRANCH https://github.com/openstack/tripleo-heat-templates
  git clone https://github.com/openstack-infra/tripleo-ci.git

  ./tripleo-ci/scripts/tripleo.sh --all

  # The last command will execute:
  #  repo_setup        --repo-setup
  #  undercloud        --undercloud
  #  overcloud_images  --overcloud-images
  #  register_nodes    --register-nodes
  #  introspect_nodes  --introspect-nodes
  #  overcloud_deploy  --overcloud-deploy
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

<div style="font-size:10px">
  <blockquote>
    <p><strong>Updated 2017/02/23:</strong> instack-virt-setup is deprecatred :( moving to tripleo-quickstart.</p>
    <p><strong>Updated 2016/11/25:</strong> instack-virt-setup env. vars. are defaulted to sane defaults, so they are optional now.</p>
  </blockquote>
</div>
