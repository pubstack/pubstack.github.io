---
layout: post
title: "Multihost deployments with Kubeinit"
author: "Carlos Camacho"
categories:
  - blog
tags:
- kubernetes
- kubeinit
- ovn
favorite: false
commentIssueId: 80
refimage: '/static/kubeinit/net/multihost.png'
---

Until now there was possible to deploy Kubeinit in
a single host configuration, this means to deploy
all the guest VMs in the same hypervisor.
Now, we can decouple the deployment architecture in multiple
hosts.

>  __*Note 2021/10/13:*__ DEPRECATED - This tutorial only works with
[kubeinit 1.0.2](https://github.com/Kubeinit/kubeinit/releases/tag/1.0.2) make
sure you use this version of the code if you are following this tutorial, or
[refer to the documentation](https://docs.kubeinit.org/) to use the latest code.

# TL;DR;

This post will show how to adjust the inventory files to
deploy Kubeinit in multiple hosts.

### Network architecture

From the official docs,
OVN (Open Virtual Network) is a series of daemons for the Open vSwitch that translate
virtual network configurations into OpenFlow.
OVN is licensed under the open source Apache 2 license.

OVN provides a higher-layer of abstraction than Open vSwitch,
working with logical routers and logical switches, rather than flows.
OVN is intended to be used by cloud management software (CMS).

Open vSwitch is a free and open source multi-layer software switch,
which is used to manage the traffic between virtual machines and
physical or logical networks.

The following is the current network architecture of a Kubeinit deployment.

![](/static/kubeinit/net/ovn.png)

### Adjusting the inventory files

To deploy Kubeinit in multiple hosts the only
changes that needs to be made are in the inventory

In this case by default there is only one hypervisor enabled (nyctea).

```bash
[hypervisor_nodes]
hypervisor-01 ansible_host=nyctea
# hypervisor-02 ansible_host=tyto
# hypervisor-03 ansible_host=strix
# hypervisor-04 ansible_host=otus
```

The only action that needs to be taken is uncomment the lines to
enable the extra hosts required.

The next step is to determine where to deploy each guest.
In this example all guests are deployed to hypervisor-01,
make the adjustments as required.

```bash
[okd_master_nodes]
okd-master-01 ansible_host=10.0.0.1 mac=52:54:00:34:84:26 interfaceid=47f2be09-9cde-49d5-bc7b-76189dfcb8a9 target=hypervisor-01 type=virtual
okd-master-02 ansible_host=10.0.0.2 mac=52:54:00:53:75:61 interfaceid=fb2028cf-dfb9-4d17-827d-3fae36cb3e98 target=hypervisor-01 type=virtual
okd-master-03 ansible_host=10.0.0.3 mac=52:54:00:96:67:20 interfaceid=d43b705e-86ce-4955-bbf4-3888210af82e target=hypervisor-01 type=virtual
```

For example, let's deploy each master node in a different hypervisor:


```bash
[okd_master_nodes]
okd-master-01 ansible_host=10.0.0.1 mac=52:54:00:34:84:26 interfaceid=47f2be09-9cde-49d5-bc7b-76189dfcb8a9 target=hypervisor-01 type=virtual
okd-master-02 ansible_host=10.0.0.2 mac=52:54:00:53:75:61 interfaceid=fb2028cf-dfb9-4d17-827d-3fae36cb3e98 target=hypervisor-02 type=virtual
okd-master-03 ansible_host=10.0.0.3 mac=52:54:00:96:67:20 interfaceid=d43b705e-86ce-4955-bbf4-3888210af82e target=hypervisor-03 type=virtual
```

Now, okd-master-01 will be deployed to hypervisor-01, okd-master-02 will be deployed to hypervisor-02,
and okd-master-03 will be deployed to hypervisor-03.

### Requirements

To deploy Kubeinit in multiple hypervisors, the only requirement is
to have passwordless root access to the hosts from the machine executing `ansible-playbook`.

### Deploying

The deployment procedure is the same
as it is for all the other Kubernetes distributions that can be
deployed with KubeInit.

Please follow the [usage documentation](http://docs.kubeinit.com/usage.html)
to understand the system's requirements and the required host supported
Linux distributions.

```bash
# Choose the distro
distro=okd

# Run the deployment command
git clone https://github.com/kubeinit/kubeinit.git
cd kubeinit
ansible-playbook \
    --user root \
    -v -i ./hosts/$distro/inventory \
    --become \
    --become-user root \
    ./playbooks/$distro.yml
```

### Conclusions

Been able to decouple the cluster across multiple hosts allows
to scale and deploy production ready environments, for different use cases.
The overlay network deployed with OVN across the hosts provides a simple abstraction
layer to communicate all the guests in the cluster.

Special thanks to
[@dalvarez](http://dani.foroselectronica.es/)
and
[@dmellado](https://github.com/danielmellado)
for their help and insights.

### The end

If you like this post, please try the code, raise issues, and ask for more details, features or
anything that you feel interested in. Also it would be awesome if you become a stargazer to catch up
updates and new features.

This is the main project [repository](https://github.com/kubeinit/kubeinit).

Happy KubeIniting!
