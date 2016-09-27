---
layout: post
title: "Deployment tips for puppet-tripleo changes"
author: "Carlos Camacho"
date: 2016-09-29 9:00:00
categories:
  - blog
tags:
  - tripleo
  - openstack
commentIssueId: 16
---

This post will describe different ways of debugging puppet-tripleo changes.

## Deploying puppet-tripleo using gerrit patches or source code repositories

In some cases, dependencies should be merged first in order to test newer
patches when adding new features to THT. With the following procedure, the user
will be able to create the overcloud images using WorkInProgress patches from
gerrit code review without having them merged (for CI testing purposes).

If using third party repos included in the overcloud image, like i.e. the
puppet-tripleo repository, your changes will not be available by default in the
overcloud until you write them in the overcloud image (by default is:
overcloud-full.qcow2)

In order to make ~~quick~~ changes to the overcloud image for testing purposes, you
can:

Export the paths to your submission by following an
[In-Progress review](http://tripleo.org/developer/in_progress_review.html):

```bash
  export DIB_INSTALLTYPE_puppet_tripleo=source
  export DIB_REPOLOCATION_puppet_tripleo=https://review.openstack.org/openstack/puppet-tripleo
  export DIB_REPOREF_puppet_tripleo=refs/changes/25/310725/14
```

In order to avoid noise on IRC, it is possible to clone puppet-tripleo and
apply the changes from your github account. In some cases this is particularly
useful as there is no need to update the patchset number.

```bash
  export DIB_INSTALLTYPE_puppet_tripleo=source
  export DIB_REPOLOCATION_puppet_tripleo=https://github.com/<usergoeshere>/puppet-tripleo
```

Remove previously created images from glance and from the user home folder by:

```bash
  rm -rf /home/stack/overcloud-full.*
  glance image-delete overcloud-full
  glance image-delete overcloud-full-initrd
  glance image-delete overcloud-full-vmlinuz
```

After this step the images can be recreated by executing:

```bash
  ./tripleo-ci/scripts/tripleo.sh --overcloud-images
```

## Debugging puppet-tripleo from overcloud images

For debugging purposes, it is possible to mount the overcloud .qcow2 file:

```bash
  #Install the libguest tool:
  sudo yum install -y libguestfs-tools

  #Create a temp folder to mount the overcloud-full image:
  mkdir /tmp/overcloud-full

  #Mount the image:
  guestmount -a overcloud-full.qcow2 -i --rw /tmp/overcloud-full

  #Check and validate all the changes to your overcloud image, go to /tmp/overcloud-full:
  #  For example, in this step you can go to /opt/puppet-modules/tripleo,

  #Umount the image
  sudo umount /tmp/overcloud-full
```

From the mounted image file it is also possible to run, for testing purposes,
the puppet manifests by using `puppet apply` and including your manifests:

```bash
  sudo puppet apply -v --debug --modulepath=/tmp/overcloud-full/opt/stack/puppet-modules -e "include ::tripleo::services::time::ntp"
```
