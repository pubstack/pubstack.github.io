---
layout: post
title: "The easiest and fastest way to deploy an OKD 4.5 cluster in a Libvirt/KVM host"
author: "Carlos Camacho"
categories:
  - blog
tags:
- kubernetes
- kubeinit
- cloud
- okd
favorite: true
commentIssueId: 69
refimage: '/static/kubeinit/okd-libvirt.png'
---

Long story short... **We will deploy an OKD 4.5 cluster in ~30 minutes (3 controllers, 1 to 10 workers, 1 service, and 1 bootstrap node) using one single command in around 30 minutes using a tool called [KubeInit](https://github.com/ccamacho/kubeinit).**

![](/static/kubeinit/okd-libvirt.png)

I wrote so much automation in the meantime I worked/learned/practiced in OpenStack/RHOSP/Kubernetes/Openshift/OKD
in the last 2Y, but suddenly I "lost" the machine where I hosted all these valuable code snippets.

With all this... I had to quickly invest some time to put together all that code. The first part is related to K8s/OKD
and I created a small project called KubeInit "The KUBErnetes INITiator" to share it with the world.

The first (and only for now) playbook will deploy in a single command a fully operational OKD 4.5 cluster with 3
master nodes, 1 compute nodes (configurable from 1 to 10 nodes), 1 services node, and 1 dummy bootstrap node.
The services node has installed HAproxy, Bind, Apache httpd, and NFS to host some of the external required cluster services.

![](/static/kubeinit/fast.jpg)

---

## Introduction

What is OpenShift?

> Red Hat OpenShift is an open source container application platform based on the Kubernetes container orchestrator for enterprise application development and deployment.
>
> -- <cite>https://www.openshift.com/</cite>

There are multiple ways of deploying the Community Distribution of Kubernetes that powers Red Hat OpenShift ([OKD](https://www.okd.io/)) depending on the underlying infrastructure where it will be installed. In this particular blog post, we will deploy it on top of a KVM host using Libvirt. The initial upstream support is described in the [official upstream OpenShift documentation](https://github.com/openshift/installer/tree/fcos/docs/dev/libvirt), but as you can see, it involves a high number of manual steps prone to manual errors, and most important, outdated references when the deployment workflow changes.

In this case, we will use a project based in Ansible playbooks and roles for deploying and configuring multiple Kubernetes distributions, the project is called [KubeInit](https://github.com/ccamacho/kubeinit).

## Requirements

 * RAM, depending on how many compute nodes this can go up to 384GB (the smallest amount required is around 64GB), configure the node's resources in the [inventory file](https://github.com/ccamacho/kubeinit/blob/master/hosts/okd/inventory#L8).
 * Be able to log in as `root` in the hypervisor node without using passwords (using SSH certificate authentication).
 * Reach the hypervisor node using the hostname `nyctea`, [you can change this in the inventory](https://github.com/ccamacho/kubeinit/blob/master/hosts/okd/inventory#L56) or add an entry in your `/etc/hosts` file.

## Deploy

That's it, now, let's execute the deployment command:

```bash
git clone https://github.com/ccamacho/kubeinit.git
cd kubeinit
ansible-playbook \
    --user root \
    -v -i ./hosts/okd/inventory \
    --become \
    --become-user root \
    ./playbooks/okd.yml
```

You should get something like:

```
[ccamacho@wakawaka kubeinit]$ time ansible-playbook \
                                  --user root \
                                  -i ./hosts/okd/inventory \
                                  --become \
                                  --become-user root \
                                  ./playbooks/okd.yml

Using /etc/ansible/ansible.cfg as config file
PLAY [Main deployment playbook for OKD] ********************************************
TASK [Gathering Facts] *************************************************************
ok: [hypervisor-01]
.
.
.
"NAME              STATUS   ROLES    AGE     VERSION",
"okd-master-01     Ready    master   16m     v1.18.3",
"okd-master-02     Ready    master   15m     v1.18.3",
"okd-master-03     Ready    master   12m     v1.18.3",
"okd-worker-01     Ready    worker   6m12s   v1.18.3"
]}]}}

PLAY RECAP *************************************************************************
hypervisor-01: ok=83 changed=39 unreachable=0 failed=0 skipped=6 rescued=0 ignored=3   

real 33m49.483s
user 2m30.920s
sys  0m19.678s
```

A ready to use OKD 4.5 cluster in ~30 minutes!

What you just executed should give you an operational OKD 4.5 cluster with 3 master nodes, 1 compute node (configurable from 1 to 10 nodes), 1 services node, and 1 dummy bootstrap node. The services node has installed HAproxy, Bind, Apache httpd, and NFS to host some of the external required cluster services.

Now, ssh into your hypervisor node and check the cluster status from the services machine.

```bash
ssh root@nyctea
ssh root@10.0.0.100
# This is now the service node (check the Ansible inventory for IPs and other details)
export KUBECONFIG=~/install_dir/auth/kubeconfig
oc get pv
oc get nodes
```

The root password of the services machine is [defined as a variable in the playbook](https://github.com/ccamacho/kubeinit/blob/master/playbooks/okd.yml#L54), but the public key of the hypervisor root user is deployed across all the cluster nodes, so, you should be able to connect to any node from the hypervisor machine using SSH certificate authentication.
Connect as the `root` user for the services machine (because is CentOS based) or as the `core` user to any other node (CoreOS based), using the IP addresses defined in the inventory file.

There are reasons for having this password-based access to the services node. Sometimes we need to connect to the services machine when we deploy for debugging purposes, in this case, if we don't set a password for the user we won't be able to log in using the console. Instead, for all the CoreOS nodes, once they are bootstrapped correctly/automatically there is no need to log in using the console, just wait until they are deployed to connect to them using SSH.

## Final thoughts

[KubeInit](https://github.com/ccamacho/kubeinit) is a simple and intuitive way to show to potential users and customers how easy an OpenShift (OKD) cluster can be deployed, managed, and used for any purpose they might require (production or development environments). Once they have the environment deployed then it's always easier to learn how it works, hack it, and even start contributing to the upstream community, if you are interested in this last part, please read the [contribution page](https://www.okd.io/#contribute) from the official OKD website.

All the Ansible automation is hosted in [https://github.com/ccamacho/kubeinit/](https://github.com/ccamacho/kubeinit/).

![](/static/kubeinit/happy.jpg)

---

The code is not perfect by any mean but is a good example of how to use a libvirt host to run your OKD cluster and it's incredibly
easy to improve and add other roles and scenarios.

Next steps, I'll clean all the lint nits around...

This is the GitHub repository [https://github.com/ccamacho/kubeinit/](https://github.com/ccamacho/kubeinit/).

Please if you like it, add some comments, test it, use it, hack it, break it, or become a stargazer ;)
