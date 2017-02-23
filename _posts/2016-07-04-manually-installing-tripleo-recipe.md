---
layout: post
title: "TripleO manual deployment"
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
  git clone https://github.com/openstack/tripleo-quickstart

  chmod u+x ./tripleo-quickstart/quickstart.sh
  bash ./tripleo-quickstart/quickstart.sh --install-deps
  bash ./tripleo-quickstart/quickstart.sh $VIRTHOST
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
