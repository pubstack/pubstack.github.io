---
layout: post
title: "Debugging submissions errors in TripleO CI"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
commentIssueId: 14
---

Landing upstream submissions might be hard if you are
not passing all the CI jobs that try to check that your
code actually works.

Let's assume that CI is working properly without any kind of
infra issue or without any error introduced by mistake from
other submissions. In which case, we might ending having something
like:

![](/static/gerrit_failed_jobs.png)

The first thing that we should do it's to double check the
status from all the other jobs that are actually in the TripleO
CI status page. This can be checked in the following site:

[http://tripleo.org/cistatus.html](http://tripleo.org/cistatus.html)

Also, we can get the jobs status by checking the Zuul dashboard.

[http://status.openstack.org/zuul/](http://status.openstack.org/zuul/])

Or checking the TripleO test cloud nodepool.

[http://grafana.openstack.org/dashboard/db/nodepool-tripleo-test-cloud](http://grafana.openstack.org/dashboard/db/nodepool-tripleo-test-cloud)

After checking that there are jobs passing CI let's check why our job
it's not passing correctly.

For each job the folder structure should be similar to:

```
[TXT]  console.html
[DIR]  logs/
  [DIR]  overcloud-cephstorage-0/
  [DIR]  overcloud-controller-0/
  [DIR]  overcloud-novacompute-0/
  [   ]  postci.txt.gz
  [DIR]  undercloud/
```

It's possible to check the deployment status in the `console.html` file
there you will see the result of all the deployment steps executed in
order to pass the CI job.

In case of having i.e. a failed deployment you can check `postci.txt.gz`
to get the actual standard error from the deployment.

Also from folders `overcloud-cephstorage-0`, `overcloud-controller-0` and
`overcloud-novacompute-0` you will have the content of `/var` that
will point out all the services logs.

Other useful tip might be to get all the job logs folder with wget and
crawl for a string containing the `Error` word.

```bash
#Get the CI job folder i.e. using the following URL.
wget -e robots=off -r --no-parent http://logs.openstack.org/00/000000/0/check-tripleo/gate-tripleo-ci-centos-7-ovb-ha/xxxxxx/
#Parse:
grep -iR "Error: " *
```

You will probably see there something pointing out an error, and hopefully
will give you clues about the next steps to fix them and land your submissions
as soon as possible.
