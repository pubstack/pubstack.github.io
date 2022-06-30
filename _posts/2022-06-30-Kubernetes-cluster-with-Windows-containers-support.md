---
layout: post
title: "Deploying a Kubernetes cluster with Windows containers support"
author: "Carlos Camacho"
categories:
  - blog
tags:
- kubernetes
- kubeinit
- windows
favorite: false
commentIssueId: 87
refimage: '/static/kubeinit/kubeinit_windows.png'
---

Workloads running on top of Windows based infrastructure still
represents a huge opportunity for 'the cloud', specific applications
like games development (Unity) relies heavily on the Windows ecosystem to
work and be used among developers and costumers. So forth the need to provide
a consistent cloud infrastructure for such software based on Windows.

# TL;DR;

-- This blog post is WIP --

This post will show you how to deploy a Kubernetes cluster with
Windows containers support, and what to expect from it.

## The good, the bad and the ugly

From what is available in the documentation, the support for Windows workloads
started in Kubernetes 1.5 (2017) as alfa, and as a stable build on 1.14 (2019),
but even at the moment of writing this post, all the documentation, the
container runtimes support, and the CNI connectivity supporting this type of
specific workloads is fairly limited. Before showing the actual 1-command deployment
magic, this post will through some review about what was needed and done to have this
'working'.

The next sections are an initial assessment of what is working, and how easy is to
integrate these functional components in a more fashioned and automated way using Kubeinit.

### The good

It works!!!. With some specific limitations about the container runtimes that are
supported, and the CNI plugins that have support for topologies like vxlan tunnels
you can have something working once you know what is currently working

There are a lot of resources spread in the Internet that will give you an idea of what
it should work, and in some cases how to deploy it.

## Useful links:

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

It is pretty cumbersome to catch all the steps, ordering, and combination of services that actually works.
Also the support for different distributions might lead to be forced to use i.e. container runtimes which
at the moment are just not supported.

### The ugly

While reading about what to install and how to configure I ended up with situations like the next one.
What's the difference between sdnoverlay from
[microsoft/windows-container-networking](https://github.com/microsoft/windows-container-networking/releases/download/v0.3.0/windows-container-networking-cni-amd64-v0.3.0.zip)
and winoverlay from
[containernetworking/plugins](https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-windows-amd64-v1.1.1.tgz)
remains unknown for me, they are supposed to do the same, but are plugins maintained by different folks with different names,
so it created a little bit (or too much) confusion.

We assume by default that the components that can be consumed in their "stable" releases will just work.
An example of this "not happening" is this, by default, I though that the CNI plugin winoverlay will
work "as-is" in Vanilla Kubernetes, because this same configuration is supported and working on OpenSHift and OKD.
This assumption might not be true, at the moment of writing this post (July 1st, 2022), the support for containerD and Windows compute nodes
[GitHub PR](https://github.com/containernetworking/plugins/pull/725) is merged, but not released upstream (1.1.1 does not have this change)
so whatever you pull from the releases versions will just won't work...

## Architectural considerations and deployment

Before going ahead, this section will briefly introduce Kubeinit's architectural reference
so you know ahead what is deployed where and how.

### Latest Kubeinit's support for Windows workloads

With some context of what will be deployed from the previous sections, let's go ahead
and test this awesome cool feature.

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
    -e kubeinit_spec="k8s-1-1-1-h" \
    -e kubeinit_libvirt_cloud_user_create=true \
    -e hypervisor_hosts_spec='[[ansible_host=nyctea],[ansible_host=tyto]]' \
    -e cluster_nodes_spec='[[when_group=compute_nodes,os=windows]]' \
    -e compute_node_ram_size=16777216 \
    ./kubeinit/playbook.yml
```

Once the deployment finishes, you can VNC into your Windows compute nodes firstly by
forwarding the 5900 port like:

```bash
[ccamacho@server]$ ssh root@nyctea -L 5900:127.0.0.1:5900
```
Then from your workstation VNC to 127.0.0.1.

This same configuration is deployed periodically, you can check the status of the execution in the
[periodic job page](https://ci.kubeinit.org/file/kubeinit-ci/jobs/okd-libvirt-1-1-1-h-periodic-pid-weekly-u/index.html)

### Considerations

Some considerations includes determining in the deployments the OS of the guests where
the workloads will run.

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

While it was hard to make it work at first, having Windows
workloads integrated in some of the use cases proves how easy
were Kubeinit's components to extend with new types of nodes
and new types of workloads.

This new use case enables other types of workloads that might benefit
other completely different usage of 'the cloud' we were used to see.

If you are interested in checking the PR's code, they are
[664](https://github.com/Kubeinit/kubeinit/pull/664),
[668](https://github.com/Kubeinit/kubeinit/pull/668),
[669](https://github.com/Kubeinit/kubeinit/pull/669),
[672](https://github.com/Kubeinit/kubeinit/pull/672), and
[676](https://github.com/Kubeinit/kubeinit/pull/676).

There might be still feature not working properly or connectivity issues between pods as
that is something I didn't have time to test.

## Update log:

<div style="font-size:10px">
  <blockquote>
    <p><strong>2022/07/01:</strong> Initial version.</p>
  </blockquote>
</div>

## The end

If you like this post, please try the code, raise issues, and ask for more details, features or
anything that you feel interested in. Also it would be awesome if you become a stargazer to catch up
updates and new features.

This is the main project [repository](https://github.com/kubeinit/kubeinit).

Happy KubeIniting!
