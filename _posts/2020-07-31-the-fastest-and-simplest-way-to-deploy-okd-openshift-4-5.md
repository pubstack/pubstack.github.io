---
layout: post
title: "The easiest and fastest way to deploy an OKD 4.5 cluster in a Libvirt/KVM host"
author: "Carlos Camacho"
categories:
  - blog
tags:
- kubernetes
- minikube
- cloud
- okd
favorite: true
commentIssueId: 69
refimage: '/static/kubeinit/okd-libvirt.png'
---

Long story short... **A single command to deploy an OKD 4.5 cluster in ~30 minutes (3 controllers, 1 to 10 workers, 1 service, and 1 bootstrap node), forget about following endless and outdated documentation.**

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

### Requirements

 * A huge amount of RAM (the smallest amount I was able to deploy is with 64GB with a smaller configuration), configure this in the [inventory file](https://github.com/ccamacho/kubeinit/blob/master/hosts/okd/inventory#L8).
 * Be able to log in as `root` in the hypervisor node.
 * Reach the hypervisor node using the hostname `nyctea` [you can change this in the inventory](https://github.com/ccamacho/kubeinit/blob/master/hosts/okd/inventory#L56) or add an entry in your `/etc/hosts` file.

That's it, super simple...

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

A ready to use OKD 4.5 cluster in ~30 minutes, yeah!

What you can do now is log in into your hypervisor and check the cluster status from the
service machine.

```bash
ssh root@nyctea
ssh root@10.0.0.100
# This is now the service node (check the Ansible inventory for IPs and other details)
export KUBECONFIG=~/install_dir/auth/kubeconfig
oc get pvc -n openshift-image-registry
oc get pv
oc get clusteroperator image-registry
oc get nodes
```

The root password of the services machine is [defined as a variable in the playbook](https://github.com/ccamacho/kubeinit/blob/master/playbooks/okd.yml#L54),
but the public key of the hypervisor root user is deployed across all the cluster nodes, so, you should be
able to connect to any node from the hypervisor machine using certs-based authentication.
Connect as the `root` user for the services machine (because is CentOS based) or as the `core` user to any other node (CoreOS based),
using the IP addresses defined in the inventory file.

There is some reasoning for this password based access to the services node. Sometimes we need to connect to the services machine when we deploy for debugging purposes,
in this case, if we don't set a password for the user we won't be able to login using the console. Instead, for all the CoreOS nodes, once they are bootstrapped
correctly/automatically there is no need to login using the console, just wait until they are deployed to connect to them using SSH.

![](/static/kubeinit/happy.jpg)

---

The code is not perfect by any mean but is a good example of how to use a libvirt host to run your OKD cluster and it's incredibly
easy to improve and add other roles and scenarios.

Next steps, I'll clean all the lint nits around...

This is the GitHub repository [https://github.com/ccamacho/kubeinit/](https://github.com/ccamacho/kubeinit/).

Please if you like it, add some comments, test it, use it, hack it, break it, or become a stargazer ;)
