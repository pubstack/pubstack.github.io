---
layout: post
title: "Querying haproxy data using socat from CLI"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
commentIssueId: 18
---

Currently (most users) I don't have any way to check the haproxy status in a TripleO virtual deployment (via web-browser) if not previously created some tunnels enabled for that purpose.

So let's check some haproxy data from our controller.

Check the controller IP:

```bash
nova-list
```

Connect to the controller:

```
ssh heat-admin@192.0.2.22
```

Now, we need to have installed socat:

```
sudo yum install -y socat
```

By default haproxy it's already configured to dump stats data to `/var/run/haproxy.sock`,
now let's query haproxy get some data from it:

* Show details like haproxy version, PID, current connections, session rates, tasks, among others.

```
echo "show info" | socat unix-connect:/var/run/haproxy.sock stdio
```

* Echo the stats about all frontents and backends as a csv.

```
echo "show stat" | socat unix-connect:/var/run/haproxy.sock stdio
```

* Display information about errors if there are any.

```
echo "show errors" | socat unix-connect:/var/run/haproxy.sock stdio
```

* Display open sessions.

```
echo "show sess" | socat unix-connect:/var/run/haproxy.sock stdio
```
