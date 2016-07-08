---
layout: post
title: "Openstack &amp; TripleO deployment using Inlunch"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
favorite: true
commentIssueId: 5
---

Today I'm going to speak about the first
Openstack installer I used to deploy [TripleO](http://www.tripleo.org).
[Inlunch](https://github.com/jistr/inlunch),
as its name aims it should make you
“Get an Instack environment prepared for you while
you head out for lunch.”

The steps that I'm used to run are:

* Connect to your remote server  (Your physical server) as
  root and generate the id_rsa.pub file and append it to
  the authorized_keys file.

```bash
ssh-keygen -t rsa
cd .ssh
cat id_rsa.pub >> authorized_keys
```

* Install some dependencies and clone [Inlunch](https://github.com/jistr/inlunch).

```bash
rpm -iUvh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-7.noarch.rpm
sudo yum -y install git ansible nano
git clone https://github.com/jistr/inlunch
```

* Go to the inlunch folder and edit the answer
  file to fits your needs. In the answers files
  by default it creates 6 nodes with 5GB RAM each,
  I usually change this to 3 nodes with 8GB RAM.

```bash
cd inlunch
vi answers.yml.example
```

* The last but not least [Inlunch](https://github.com/jistr/inlunch)
step it is to deploy
our undercloud!!! As simple as it sounds. As you can
see [Inlunch](https://github.com/jistr/inlunch) uses
[Ansible](http://www.ansible.com/) for all the steps automation
using SSH.
In this case we added the root public key to the same server
and the installation is pointed to localhost.

```bash
INLUNCH_ANSWERS=answers.yml.example INLUNCH_FQDN=localhost ./instack-virt.sh
```

* Once you have finished this last step you can
login in the undercloud node by sshing to the physical
server using the 2200 port.

```bash
ssh -p 2200 root@<your_server_fqdn_goes_here>
```

This is it :) your undercloud it is up and running.

Now, I will show the following steps to deploy
the master branch of tripleo-heat-templates to
finish the overcloud deployment.

* Login as the stack user and source the stackrc file

```bash
su - stack
source stackrc
```

* Let's clone all needed repositories.

```bash
git clone https://github.com/openstack/puppet-tripleo
git clone https://github.com/openstack/tripleo-docs
git clone https://github.com/openstack/tripleo-heat-templates
```

* And to finish let's deploy the [TripleO](http://www.tripleo.org) pacemaker environment.

```bash
  openstack overcloud deploy \
  --libvirt-type qemu \
  --ntp-server clock.redhat.com \
  --templates /home/stack/tripleo-heat-templates \
  -e /home/stack/tripleo-heat-templates/overcloud-resource-registry-puppet.yaml \
  -e /home/stack/tripleo-heat-templates/environments/puppet-pacemaker.yaml
```

Now you should have deployed successfully your undercloud/overcloud environment using [Inlunch](https://github.com/jistr/inlunch).

Thanks [Jiri](https://github.com/jistr/) for this amazing installer!!
