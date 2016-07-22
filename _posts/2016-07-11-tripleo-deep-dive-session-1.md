---
layout: post
title: "TripleO deep dive session #1 (Quickstart deployment)"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
commentIssueId: 6
---

This is the first video from a series of "Deep Dive" sessions
related to [TripleO](http://www.tripleo.org/) deployments.

The first session is related to the TripleO deployment using
Quickstart.

Quickstart comes from [RDO](http://www.rdoproject.org/), to reduce the complexity of having
a TripleO environment quickly, mostly for users without a strong
and deep knowledge of TripleO configuration and it uses Ansible roles
to automate all the different configuration tasks.

So please, check the full [session](https://www.youtube.com/watch?v=E1d_RmysnA8) content on the [TripleO YouTube channel](https://www.youtube.com/channel/UCNGDxZGwUELpgaBoLvABsTA/).

{% assign videoId = "E1d_RmysnA8" %}
{% include youtubePlayer.html id=videoId %}

Last but not least, James Slagle (slagle) have posted some comments about
how to apply new changes in the puppet modules when deploying the overcloud
as the current task of re-create them is a time consuming and cumbersome process.

Using the upload-puppet-modules script we will be able to update the puppet
modules when executing the overcloud deployment.

```bash
# From the undercloud 
mkdir puppet-modules
cd puppet-modules
git clone https://git.openstack.org/openstack/puppet-tripleo tripleo
# Edit as needed under the tripleo folder
cd
git clone https://git.openstack.org/openstack/tripleo-common
export PATH="$PATH:tripleo-common/scripts"
upload-puppet-modules --directory puppet-modules/

```

> Sessions index:
>
> &nbsp;&nbsp;&nbsp; * [TripleO deep dive #1 (Quickstart deployment)](http://www.anstack.com/blog/2016/07/11/tripleo-deep-dive-session-1.html)
>
> &nbsp;&nbsp;&nbsp; * [TripleO deep dive #2 (TripleO Heat Templates)](http://www.anstack.com/blog/2016/07/18/tripleo-deep-dive-session-2.html)
>
> &nbsp;&nbsp;&nbsp; * [TripleO deep dive #3 (Overcloud deployment debugging)](http://www.anstack.com/blog/2016/07/22/tripleo-deep-dive-session-3.html)

```text
---------------------------------------------------------------------------------------
|                                     ,   .   ,                                       |
|                                     )-_'''_-(                                       |
|                                    ./ o\ /o \.                                      |
|                                   . \__/ \__/ .                                     |
|                                   ...   V   ...                                     |
|                                   ... - - - ...                                     |
|                                    .   - -   .                                      |
|                                     `-.....-´                                       |
|                          _______   _       _       ____                             |
|                         |__   __| (_)     | |     / __ \                            |
|                            | |_ __ _ _ __ | | ___| |  | |                           |
|                            | | '__| | '_ \| |/ _ \ |  | |                           |
|                            | | |  | | |_) | |  __/ |__| |                           |
|                     _____  |_|_|  |_| .__/|_|\___|\____/                            |
|                    |  __ \          | |     |  __ \(_)                              |
|                    | |  | | ___  ___|_|__   | |  | |___   _____                     |
|                    | |  | |/ _ \/ _ \ '_ \  | |  | | \ \ / / _ \                    |
|                    | |__| |  __/  __/ |_) | | |__| | |\ V /  __/                    |
|                    |_____/ \___|\___| .__/_ |_____/|_| \_/ \___|                    |
|                                     | |  (_)                                        |
|                         ___  ___  __|_|__ _  ___  _ __  ___                         |
|                        / __|/ _ \/ __/ __| |/ _ \| '_ \/ __|                        |
|                        \__ \  __/\__ \__ \ | (_) | | | \__ \                        |
|                        |___/\___||___/___/_|\___/|_| |_|___/                        |
|                                                                                     |
---------------------------------------------------------------------------------------
```

