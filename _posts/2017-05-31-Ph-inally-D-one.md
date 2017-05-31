---
layout: post
title: "Ph.inally D.one!"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - reflections
  - draft
#commentIssueId:
---




<div style="float: left; width: 250px; background: red;"><img src="/static/bandaid.jpg" alt=""></div>

If running ercloud or Overcloud nodes and
getting some output like:

And as in the example there is no reference pointing
to the swap memory size and/or usage, you might not be using swap
in your TripleO deployments, to enable it, just have
to follow two steps.

<div style="float: right; width: 250px; background: red;"><img src="/static/bandaid.jpg" alt=""></div>
    
First in the Undercloud, when deploying stacks you might find
that heat-engine (4 workers) takes lot of RAM, in this
case for specific usage peaks can be useful to have a
swap file. In order to have this swap file enabled and used by the OS
execute the following instructions in the Undercloud:
Also when deploying the Overcloud nodes the controller might face
some RAM usage peaks, in which case, create a swap file in each
Overcloud node by using an already existing “extraconfig swap”
template.

To achieve this second part, we just need to use the environmental
file that loads the swap template in the resource

![](/static/openstack-summit-2016-bcn.jpeg)

