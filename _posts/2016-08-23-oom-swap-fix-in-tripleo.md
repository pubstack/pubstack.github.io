---
layout: post
title: "BAND-AID for OOM issues with TripleO manual deployments"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
commentIssueId: 13
---
![](/static/bandaid.jpg)

If running `free -m` does not show something pointing
to the swap memory usage, you might not be using swap
in your TripleO deployments, to enable it, just have
to follow two steps..

First in the Undercloud, when deploying stacks you might find
that heat-engine (4 workers) takes lot of RAM, in this
case for specific usage peaks can be useful to have a
swap file like (Undercloud):

```bash
#Add a 4GB swap file to the Undercloud
sudo dd if=/dev/zero of=/swapfile bs=1024 count=4194304
sudo mkswap /swapfile
#Turn ON the swap file
sudo chmod 600 /swapfile
sudo swapon /swapfile
#Enable it on start
echo "/swapfile swap swap defaults 0 0" | sudo tee -a /etc/fstab
```

Also when deploying the Overcloud nodes the controller might face
some RAM usage peaks, in which case, create a swap file in each
Overcloud node by using the extraconfig swap template:

```bash
cd
openstack overcloud deploy \
--libvirt-type qemu \
--ntp-server pool.ntp.org \
--templates /home/stack/tripleo-heat-templates \
-e /home/stack/tripleo-heat-templates/overcloud-resource-registry-puppet.yaml \
-e /home/stack/tripleo-heat-templates/environments/puppet-pacemaker.yaml \
-e /home/stack/tripleo-heat-templates/extraconfig/all_nodes/swap.yaml #This one!!
```

The last two hints will provide a swap file of 4GB in
both Undercloud and Overcloud nodes.
Having this swap file might be the difference between
decreasing the system performance because of the
high memory usage or a OOM deployment crash.
