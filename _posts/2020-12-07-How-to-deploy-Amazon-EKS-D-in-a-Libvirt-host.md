---
layout: post
title: "How to deploy Amazon EKS-D on top of a Libvirt host with KubeInit in 15 minutes"
author: "Carlos Camacho"
categories:
  - blog
tags:
- kubernetes
- kubeinit
- cloud
- eks
- eks-d
favorite: false
commentIssueId: 79
refimage: '/static/kubeinit/eks-d.png'
---

And there is a new distro in town, today we will speak about
Amazon EKS Distro (EKS-D), a Kubernetes distribution based on
Amazon Elastic Kubernetes Service (Amazon EKS)
and how to deploy it in a Libvirt host with almost or zero
effort in a few minutes.

>  __*Note 2021/10/13:*__ DEPRECATED - This tutorial only works with
[kubeinit 1.0.2](https://github.com/Kubeinit/kubeinit/releases/tag/1.0.2) make
sure you use this version of the code if you are following this tutorial, or
[refer to the documentation](https://docs.kubeinit.org/) to use the latest code.

# TL;DR;

We will deploy using KubeInit a Kubernetes cluster based in Amazon's EKS distribution.
**Disclaimer:** This is not completely implemented as there are still some EKS-D images
to be added to the deployment.

### Components

Here is a list of the components that are currently deployed:

* Guests OS: CentOS 8 (8.2.2004)
* Kubernetes distribution: EKS-D
* Infrastructure provider: Libvirt
* A service machine with the following services:
    - HAProxy: 1.8.23 2019/11/25
    - Apache: 2.4.37
    - NFS (nfs-utils): 2.3.3
    - DNS (bind9): 9.11.13
    - Disconnected docker registry: v2
    - Skopeo: 0.1.40
* Control plane services:
    - Kubelet 1.18.4
    - CRI-O: 1.18.4
    - Podman: 1.6.4
* Controller nodes: 3
* Worker nodes: 1

### Deploying

The deployment procedure is the same
as it is for all the other Kubernetes distributions that can be
deployed with KubeInit.

**Note:** Make sure you can connect to your hypervisor (called nyctea)
with passwordless access.

Please follow the [usage documentation](http://docs.kubeinit.com/usage.html)
to understand the system's requirements and the required host supported
Linux distributions.

```bash
# Choose the distro
distro=eks

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

You can also [run it from a container](https://www.pubstack.com/blog/2020/09/11/Deploying-KubeInit-from-a-container.html)
to avoid compatibility issues between your set up and the required libraries.

This will deploy by default a 3 controllers 1 compute cluster.

The deployment time was fairly quick (around 15 minutes):

```bash
.
.
.
  "      description: snapshot-validation-webhook container image",
  "      image:",
  "        uri: public.ecr.aws/eks-distro/kubernetes-csi/external-snapshotter/snapshot-validation-webhook:v3.0.2-eks-1-18-1",
  "      name: snapshot-validation-webhook-image",
  "      os: linux",
  "      type: Image",
  "    gitTag: v3.0.2",
  "    name: external-snapshotter",
  "  date: \"2020-12-01T00:05:35Z\""
]
}
META: ran handlers
META: ran handlers

PLAY RECAP *****************************************************************************************************************
hypervisor-01              : ok=188  changed=93   unreachable=0    failed=0    skipped=43   rescued=0    ignored=4   

real	17m12.889s
user	1m24.846s
sys	0m24.366s
```

Let's run some commands in the cluster.

```bash
[root@eks-service-01 ~]# curl --user registryusername:registrypassword https://eks-service-01.clustername0.kubeinit.local:5000/v2/_catalog
{
   "repositories":[
      "aws-iam-authenticator",
      "coredns",
      "csi-snapshotter",
      "eks-distro/coredns/coredns",
      "eks-distro/etcd-io/etcd",
      "eks-distro/kubernetes/go-runner",
      "eks-distro/kubernetes/kube-apiserver",
      "eks-distro/kubernetes/kube-controller-manager",
      "eks-distro/kubernetes/kube-proxy",
      "eks-distro/kubernetes/kube-proxy-base",
      "eks-distro/kubernetes/kube-scheduler",
      "eks-distro/kubernetes/pause",
      "eks-distro/kubernetes-csi/external-attacher",
      "eks-distro/kubernetes-csi/external-provisioner",
      "eks-distro/kubernetes-csi/external-resizer",
      "eks-distro/kubernetes-csi/external-snapshotter/csi-snapshotter",
      "eks-distro/kubernetes-csi/external-snapshotter/snapshot-controller",
      "eks-distro/kubernetes-csi/external-snapshotter/snapshot-validation-webhook",
      "eks-distro/kubernetes-csi/livenessprobe",
      "eks-distro/kubernetes-csi/node-driver-registrar",
      "eks-distro/kubernetes-sigs/aws-iam-authenticator",
      "eks-distro/kubernetes-sigs/metrics-server",
      "etcd",
      "external-attacher",
      "external-provisioner",
      "external-resizer",
      "go-runner",
      "kube-apiserver",
      "kube-controller-manager",
      "kube-proxy",
      "kube-proxy-base",
      "kube-scheduler",
      "livenessprobe",
      "metrics-server",
      "node-driver-registrar",
      "pause",
      "snapshot-controller",
      "snapshot-validation-webhook"
   ]
}
```

And check some of the deployed resources.

```bash
[root@eks-service-01 ~]# kubectl describe pods etcd-eks-master-01.kubeinit.local -n kube-system
Name:                 etcd-eks-master-01.kubeinit.local
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 eks-master-01.kubeinit.local/10.0.0.1
Start Time:           Sun, 06 Dec 2020 19:51:25 +0000
Labels:               component=etcd
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.0.0.1:2379
                      kubernetes.io/config.hash: 3be258678a84985dbdb9ae7cb90c6a97
                      kubernetes.io/config.mirror: 3be258678a84985dbdb9ae7cb90c6a97
                      kubernetes.io/config.seen: 2020-12-06T19:51:18.652592779Z
                      kubernetes.io/config.source: file
Status:               Running
IP:                   10.0.0.1
IPs:
  IP:           10.0.0.1
Controlled By:  Node/eks-master-01.kubeinit.local
Containers:
  etcd:
    Container ID:  cri-o://7a52bd0b80feb8c861c502add4c252e83c7e4a1f904a376108e3f6f787fd342c
    Image:         eks-service-01.clustername0.kubeinit.local:5000/etcd:v3.4.14-eks-1-18-1
```

### Conclusions

Either if this new distribution can be useful or not for your use cases, there is
for sure value on having in both cloud and on-premise the same architecture and
services consistency.

### The end

If you like this post, please try the code, raise issues, and ask for more details, features or
anything that you feel interested in. Also it would be awesome if you become a stargazer to catch up
updates and new features.

This is the main project [repository](https://github.com/kubeinit/kubeinit).

Happy KubeIniting!
