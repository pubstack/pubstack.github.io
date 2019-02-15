---
layout: post
title: "Connecting from your local machine to
the TripleO overcloud horizon dashboard"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
  - cloud
favorite: true
commentIssueId: 1
refimage: '/static/multi-hop.png'
---

This will be my first blog post about TripleO deployments.

The goal of this post is to show how to chain multiple ssh
tunnels to browse into the horizon dashboard, deployed
in a TripleO environment.

In this case, we have deployed [TripleO](http://www.tripleo.org/)
in a remote server "labserver" in which was launched an undercloud
and overcloud deployment.

The horizon dashboard listens on the 80 port of the overcloud controller.
From the user terminal we want to have access to the horizon dashboard, currently
unreachable because we don't have access to the deployed private IPs from the
user's terminal.

Below is a graphical representation about the described scenario.
![](/static/multi-hop.png)    

## STEPS
* Connect the local terminal to labserver (create the first tunnel)

```bash
#Forward incoming 38080 traffic to local 38080 in the hypervisor
#labserver must be a reachable host
ssh -L 38080:localhost:38080 root@labserver
```

* Connect to the undercloud from the labserver (create the second tunnel)

```bash
#Log-in as the stack user and get the undercloud IP
su - stack
undercloudIp=`sudo virsh domifaddr instack | grep $(tripleo get-vm-mac instack) | awk '{print $4}' | sed 's/\/.*$//'`
#Forward incoming 38080 traffic to local 38080 in the undercloud
ssh -L 38080:localhost:38080 root@$undercloudIp
```

* Get the admin password for the Horizon dashboard

```bash
su - stack
source stackrc
cat overcloudrc |grep OS_PASSWORD | awk -F  '=' '{print $2}'
```

* Connect to the overcloud controller from the undercloud (create the third and last tunnel)

```bash
#Get the controller IP
controllerIp=`nova list | grep controller | awk -F  '|' '{print $7}' | awk -F  '=' '{print $2}'`
#Forward incoming 38080 traffic to controller IP in 80 port
ssh -L 38080:"$controllerIp":80 heat-admin@"$controllerIp"
echo "From your browser open: http://localhost:38080/"
```

Now, if everything went as expected you should be able to see
the horizon dashboard by using your favorite browser and typing http://localhost:38080/dashboard,  
then use the admin user and the password printed before.

Note that in this case, by default the ssh hops should be done using different users.
