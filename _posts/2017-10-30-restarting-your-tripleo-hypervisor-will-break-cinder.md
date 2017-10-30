---
layout: post
title: "Restarting your TripleO hypervisor will break cinder volume service thus the overcloud pingtest"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
commentIssueId: 39
---

I don't usualy restart my hypervisor, today I had to install LVM2 and
virsh stopped to work so a restart was required, once the VMs were
up and running the overcloud pingtest failed as cinder was not able to start.

From your Overcloud controller run: 

```
sudo losetup -f /var/lib/cinder/cinder-volumes
sudo vgdisplay
sudo service openstack-cinder-volume restart
```

This will make your Overcloud pingtest work again.
