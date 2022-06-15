---
layout: post
title: "TripleO cheatsheet"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
  - cloud
favorite: true
commentIssueId: 22
refimage: '/static/tripleo_banner.png'

---

This is a cheatsheet some of my regularly used
commands to test, develop or debug
TripleO deployments.

<p>Deployments</p>

<ul>
  <li>
    <code class="highlighter-rouge">
      swift download overcloud
    </code><br>
    <p class="tdesc">
        Download the <code class="highlighter-rouge">overcloud</code> swift container files in the current folder (With the rendered j2 templates).
    </p>
  </li>
  <li>
    <code class="highlighter-rouge">
      heat resource-list --nested-depth 5 overcloud | grep FAILED
    </code><br>
    <p class="tdesc">
      Show resources, filtering to get those who have failed.
    </p>
  </li>
  <li>
    <code class="highlighter-rouge">
      heat deployment-show &lt;deployment_ID&gt;
    </code><br>
    <p class="tdesc">
      Get the deployment details for &lt;deployment_ID&gt;.
    </p>
  </li>
  <li>
    <code class="highlighter-rouge">
      openstack image list
    </code><br>
    <p class="tdesc">
      List images.
    </p>
  </li>
  <li>
    <code class="highlighter-rouge">
         openstack image delete &lt;image_ID&gt;
    </code><br>
    <p class="tdesc">
      Delete &lt;image_ID&gt;.
    </p>
  </li>
  <li>
    <code class="highlighter-rouge">
     wget http://buildlogs.centos.org/centos/7/cloud/x86_64/tripleo_images/&lt;release&gt;/delorean/overcloud-full.tar
    </code><br>
    <p class="tdesc">
      Download &lt;release&gt; overcloud images tar file [liberty|mitaka|newton|...]
    </p>
  </li>
  <li>
    <code class="highlighter-rouge">
      openstack overcloud image upload --update-existing
    </code><br>
    <p class="tdesc">
      Once downloaded the images, this command will upload them to glance.
    </p>
  </li>
</ul>

<p>Debugging CI</p>

<ul>
  <li>
    <code class="highlighter-rouge">
      http://status.openstack.org/zuul/
    </code><br>
    <p class="tdesc">
      Check submissions CI status.
    </p>
  </li>
  <li>
    <code class="highlighter-rouge">
      wget -e robots=off -r --no-parent &lt;patch_ID&gt;
    </code><br>
    <p class="tdesc">
      Download all logs from &lt;patch_ID&gt;.
    </p>
  </li>
  <li>
    <code class="highlighter-rouge">
      console.html
    </code>
    &amp;
    <code class="highlighter-rouge">
      logs/postci.txt.gz
    </code><br>
    <p class="tdesc">
      Relevant log files when debuging a TripleO Gerrit job.
    </p>
  </li>
</ul>

> If you think there are more
> useful commands to add to the list
> just add a [comment](https://github.com/pubstack/pubstack.github.io/issues/22)!

**Happy TripleOing!**

{% comment %}
 To add:
{% endcomment %}
