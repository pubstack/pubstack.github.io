---
layout: post
title: "Testing instack-undercloud submissions locally"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
  - instack-undercloud
commentIssueId: 9
---

This post is to describe how to run/test gerrit submissions related to instack-undercloud locally.

For this example I'm going to use this submission: https://review.openstack.org/#/c/347389/

The follwing steps allow to test the submissions related to instack-undercloud
in a working environment.

```bash
  ./tripleo-ci/scripts/tripleo.sh --delorean-setup
  ./tripleo-ci/scripts/tripleo.sh --delorean-build openstack/instack-undercloud
  cd tripleo/instack-undercloud/
  #The submission to be tested
  git review -d 347389
  cd
  ./tripleo-ci/scripts/tripleo.sh --delorean-build openstack/instack-undercloud
  rpm -qa | grep instack-undercloud
  sudo rpm -e --nodeps <old_installed_instack-undercloud>
  find tripleo/ -name "*rpm"
  sudo rpm -iv --replacepkgs --force <located package>
  #Here we need to check that the changes were actually applied.
  #What I'm used to do it's to search the updated files using locate
  #and manually checking that the changes are OK.
  sudo rm -rf /root/.cache/image-create/source-repositories/*
  sudo rm -rf /opt/stack/puppet-modules
```

Now, in case that a puppet-tripleo change is needed, you can add the env. vars before
re-installing the undercloud.

```bash
  export DIB_INSTALLTYPE_puppet_tripleo=source
  export DIB_REPOLOCATION_puppet_tripleo=https://review.openstack.org/openstack/puppet-tripleo
  export DIB_REPOREF_puppet_tripleo=refs/changes/XX/XXXXX/X
```

Now, we just need to run the installer.

```bash
  ./tripleo-ci/scripts/tripleo.sh --undercloud
```

Once this process completes, the output should be something similar to:

```text

#################
tripleo.sh -- Undercloud install - DONE.
#################

```
