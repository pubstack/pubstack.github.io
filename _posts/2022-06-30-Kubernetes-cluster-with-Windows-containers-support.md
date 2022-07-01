---
layout: post
title: "Deploying a Kubernetes cluster with Windows containers support"
author: "Carlos Camacho"
categories:
  - blog
tags:
- cloud
- kubernetes
- kubeinit
- windows
favorite: true
commentIssueId: 87
refimage: '/static/kubeinit/kubeinit_windows.png'
---

Workloads running on top of Windows-based infrastructure still
represent a huge opportunity for 'the cloud', specific applications
like video-games development (Unity) rely heavily on the Microsoft Windows ecosystem
to work and be used among developers and customers. Hence the need
to provide a consistent cloud infrastructure for such Windows based software.

# TL;DR;

This post will show you how to deploy a Kubernetes cluster with
Windows containers support, and what to expect from it.

## The good, the bad and the ugly

From what is available in the documentation, the support for Windows workloads
started in Kubernetes 1.5 (2017 'alfa'), and as a stable build on K8s 1.14 (2019),
but even at the moment of writing this post, all the documentation, the
container runtime support, and the CNI connectivity supporting this type of
specific workloads is fairly limited. Before showing the actual 1-command deployment
magic, this post will go through an brief review about what was needed, and what was
done to have this 'working'.

The following sections are an initial assessment of what is working, and how easy is to
integrate these functional components in a more fashioned and automated way using Kubeinit.

### The good

It works!!!. With some specific limitations about the container runtimes that are
supported, and the CNI plugins that have support for topologies like vxlan tunnels
you can have something working once you know what is currently supported for your distro.

There are a lot of resources spread on the Internet that will give you an idea of what
should work, and in some cases how-to deploy it.

#### Useful links:

This is the partial list of resources checked to finish the integration of Windows workloads
in Kubeinit:

Blog posts and documentation:
* https://www.jamessturtevant.com/posts/Windows-Containers-on-Windows-10-without-Docker-using-Containerd/
* https://github.com/lippertmarkus/vagrant-k8s-win-hostprocess
* https://docs.microsoft.com/en-us/virtualization/windowscontainers/kubernetes/common-problems
* https://techcommunity.microsoft.com/t5/networking-blog/introducing-kubernetes-overlay-networking-for-windows/ba-p/363082
* https://deepkb.com/CO_000014/en/kb/IMPORT-4bb99d54-1582-32fa-b130-b496089f7678/guide-for-adding-windows-nodes-in-kubernetes

Official code from Kubernetes:
* https://github.com/kubernetes/kubernetes/issues/94924
* https://github.com/kubernetes/kubernetes/blob/master/cluster/gce/windows/k8s-node-setup.psm1

Code from the CNI Microsoft team:
* https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/start-kubelet.ps1
* https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1
* https://github.com/microsoft/SDN/tree/master/Kubernetes/flannel/overlay
* https://github.com/microsoft/SDN/blob/master/Kubernetes/flannel/register-svc.ps1

Code from the Windows K8s sig:
* https://github.com/kubernetes-sigs/sig-windows-dev-tools
* https://github.com/kubernetes-sigs/sig-windows-tools/issues/128

Code from the CNI dev team:
* https://github.com/containernetworking/plugins/blob/main/plugins/main/windows/win-overlay/sample-v2.conf

### The bad

It is pretty cumbersome to catch all the steps, ordering, and combinations of services
that actually work. Also, the support for different distributions might lead to being
forced to use i.e. container runtimes which at the moment are just not supported.

### The ugly

While reading about what to install and how to configure I ended up with situations like the next one.
What's the difference between sdnoverlay from
[microsoft/windows-container-networking](https://github.com/microsoft/windows-container-networking/releases/download/v0.3.0/windows-container-networking-cni-amd64-v0.3.0.zip)
and winoverlay from
[containernetworking/plugins](https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-windows-amd64-v1.1.1.tgz)
remains unknown to me (mostly because of my limited time to dig into it with more detail), they are supposed
to do the same, but there are plugins maintained by different folks with different names,
so it created a little bit (or too much) confusion for me when doing this initial integration.

We assume by default that the components that can be consumed in their "stable" releases will just work.
An example of this "not happening" is this, by default, I thought that the CNI plugin winoverlay will
work "as-is" in Vanilla Kubernetes, because this same configuration is supported and working on OpenSHift and OKD.
This assumption might not be true, at the moment of writing this post (July 1st, 2022), the support for containerD and Windows compute nodes in this
[GitHub PR](https://github.com/containernetworking/plugins/pull/725) is merged, but not released upstream (v1.1.1 does not have this change)
so whatever you pull from the released versions will just won't work...

## Architectural considerations and deployment

Before going ahead, this section will briefly introduce Kubeinit's architectural reference
so you know ahead what is deployed, where, and how.

![](/static/kubeinit/arch/kubeinit_network_legend.png)
The picture above shows the legend of the main functional components of the platform.

The left icon represents the services pod, that will run all the infrastructure services required to have the cluster up and running, services like HAProxy, Bind, and a local container registry among others that will support the cluster external required services.

The right icon represents the Virtual Machines instances that will host each node of the cluster, these nodes can be, the control-plane nodes and the worker nodes, the control-plane nodes will be Linux, and depending on the Kubernetes distribution they will have installed CentOS Stream, Debian, Fedora CoreOS, or CoreOS. The worker nodes will have the same Linux distribution as the control-plane nodes with the addition of the recently added Windows worker nodes.

![](/static/kubeinit/arch/kubeinit_network_physical.png)

Now, we have a representation of the physical topology of a deployed cluster, the requirement is to have a set of hypervisors where the cluster nodes (guests) will be deployed, these hypervisors must be connected in a way that they can all reach each others (or having them connected in a L2 segment). Another assumed-by-default requirement is to have
SSH passwordless access to the nodes where you are calling Ansible from.

![](/static/kubeinit/arch/kubeinit_network_logical.png)

The logical topology represents a fairly more detailed view about how the components are actually 'connected'
by allowing all the guests in the cluster (including the services pod) to be reachable without any distinction
of what is deployed where within the cluster.

In this particular case, we have installed OVS in each hypervisor and by using OVN we create an internal overlay
network to provide a consistent and uniform way to access any cluster resource.

### Latest Kubeinit's support for Windows workloads

With some context of what will be deployed from the previous sections, let's go ahead
and test this awesome cool feature in a magical 1-command deployment.

> NOTE: Please check the complete instructions from the main
> [README](https://github.com/Kubeinit/kubeinit#readme) page,
> also, a good reference to know the hypervisor requirements is the
> [CI install script](https://github.com/Kubeinit/kubeinit/blob/main/ci/install_gitlab_node.sh)
> there you will find what is required to set up the hypervisors based on Debian/Ubuntu/CentOS Stream/Fedora.

### Deploying

```bash
# Install the requirements assuming python3/pip3 is installed
pip3 install \
        --upgrade \
        pip \
        shyaml \
        ansible \
        netaddr

# Get the project's source code
git clone https://github.com/Kubeinit/kubeinit.git
cd kubeinit

# Install the Ansible collection requirements
ansible-galaxy collection install --force --requirements-file kubeinit/requirements.yml

# Build and install the collection
rm -rf ~/.ansible/collections/ansible_collections/kubeinit/kubeinit
ansible-galaxy collection build kubeinit --verbose --force --output-path releases/
ansible-galaxy collection install --force --force-with-deps releases/kubeinit-kubeinit-`cat kubeinit/galaxy.yml | shyaml get-value version`.tar.gz

# Run the deployment
ansible-playbook \
    --user root \
    -v \
    -e kubeinit_spec="k8s-libvirt-1-1-1" \
    -e kubeinit_libvirt_cloud_user_create=true \
    -e hypervisor_hosts_spec='[[ansible_host=nyctea],[ansible_host=tyto]]' \
    -e cluster_nodes_spec='[[when_group=compute_nodes,os=windows]]' \
    -e compute_node_ram_size=16777216 \
    ./kubeinit/playbook.yml
```

If you will like to cleanup your environment after using Kubeinit, just run the same
deployment command appending `-e kubeinit_stop_after_task=task-cleanup-hypervisors`
that will clean all the resources deployed by the installer.

> NOTE: If you have an error when deploying the services pod like
> 'TASK [kubeinit.kubeinit.kubeinit_services : Install python3] ... unreachable Could not resolve host: mirrorlist.centos.org'
> make sure your DNS server is reachable, by default it is used the 1.1.1.1
> DNS server from Google, and it might be blocked in your internal network, run
> `export KUBEINIT_COMMON_DNS_PUBLIC=<your valid DNS>` and
> then run the deployment as usual.

The 'new' parameter present when running the Ansible playbook is called
`cluster_nodes_spec`, this parameter allows to determine
the OS (Operative System) of the compute nodes. There are still in progress some features
to fully allow to customize the setup and decide how many nodes will be Windows based and how many will
have Linux installed.
The only supported version is Windows server 2022 Datacenter Edition.
This Windows Server installation is based on the actual .ISO installer, so if a user want's
to use this in a more stable scenario they will need to register these cluster's nodes.

Once the deployment finishes, in the case of having Windows compute nodes it
will be around ~50 minutes (at least the first time because we need to download all the .ISO images),
you can VNC into your Windows compute nodes firstly by forwarding the 5900 port like:

```bash
[ccamacho@laptop]$ ssh root@nyctea -L 5900:127.0.0.1:5900
```

Then, from your workstation start a VNC session to 127.0.0.1:5900.

Or even better, you can SSH directly into your Windows computes like:

```bash
# This will get you into the first controller node.
[ccamacho@nyctea]$ ssh -i .ssh/k8scluster_id_rsa root@10.0.0.1
# This will get you to your first compute node.
[ccamacho@nyctea]$ ssh -i .ssh/k8scluster_id_rsa root@10.0.0.2
# This will get you to the services pod.
[ccamacho@nyctea]$ ssh -i .ssh/k8scluster_id_rsa root@10.0.0.253
```

The default IP segment assigned for the cluster nodes is the 10.0.0.0/24 network,
where  the IPs are assigned in order, first the controller nodes, then the computes, and the last
IP for the services pod. In this example, given that the kubeinit_spec is `k8s-libvirt-1-1-1`
we will deploy a vanilla Kubernetes cluster on top of libvirt with one controller, one compute,
and using a single hypervisor (explained in the same order than the spec content).

This same configuration is deployed periodically, you can check the status of the execution in the
[periodic job page](https://ci.kubeinit.org/file/kubeinit-ci/jobs/okd-libvirt-1-1-1-h-periodic-pid-weekly-u/index.html)

### Considerations

After having the cluster deployed, some considerations include determining from now on
in the applications deployments the OS of the guests where the workloads will run.

The behavior I was able to see is that the Linux deployment tried to be scheduled in the
Windows compute nodes, which at some point it just timed-out.

So make sure your deployments use the nodeSelector like:

```bash
nodeSelector:
 kubernetes.io/os: linux
```

or

```bash
nodeSelector:
 kubernetes.io/os: windows
```

## Conclusions

While it was hard to make it work at first, having
Windows workloads integrated into some of the use cases,
proves how easy it was to extend Kubeinit's core architectural
structure with new types of nodes, and new types of workloads.

This new use case enables other types of workloads that might
benefit other completely different usages of 'the cloud' we were used to seeing.

If you are interested in checking the PR's code, they are
[664](https://github.com/Kubeinit/kubeinit/pull/664),
[668](https://github.com/Kubeinit/kubeinit/pull/668),
[669](https://github.com/Kubeinit/kubeinit/pull/669),
[672](https://github.com/Kubeinit/kubeinit/pull/672), and
[676](https://github.com/Kubeinit/kubeinit/pull/676).

There might be still features not working properly or connectivity issues between pods as that is something I didn't have time to properly
test so far.
The architectural [diagrams](https://drive.google.com/file/d/1l9AHJ_60SNWYFVPWgO4C__U0bwPJvVIT/view?usp=sharing) are available
online for further edits and references.

## Update log:

<div style="font-size:10px">
  <blockquote>
    <p><strong>2022/07/01:</strong> Initial version and minor edits.</p>
  </blockquote>
</div>

## The end

If you like this post, please try the code, raise issues, and ask for more details, features or
anything that you feel interested in. Also it would be awesome if you become a stargazer to catch up
updates and new features.

This is the project's main [repository](https://github.com/kubeinit/kubeinit).

Happy KubeIniting!
