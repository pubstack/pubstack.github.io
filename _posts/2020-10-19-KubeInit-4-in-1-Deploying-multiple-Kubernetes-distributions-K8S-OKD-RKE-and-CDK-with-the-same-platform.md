---
layout: post
title: "KubeInit 4-in-1 - Deploying multiple Kubernetes distributions (K8S, OKD, RKE, and CDK) with the same platform"
author: "Carlos Camacho"
categories:
  - blog
tags:
- kubernetes
- kubeinit
- cloud
- okd
favorite: false
commentIssueId: 77
refimage: '/static/kubeinit/kubeinit-4-in-1.png'
---

One of the KubeInit's pillars is to define a common framework to deploy
multiple Kubernetes distributions, once you finish the deployment you should
be able to use the specific distro tooling to mange the lifecycle of your deployment
(or multiple deployments).

Using KubeInit you should be able to reuse a common set of third party services,
and infrastructure deployment assets with any already integrated distro.

The current distributions of Kubernetes that should be deployable are:

* Origin Kubernetes Distribution
* Kubernetes
* Rancher Kubernetes Distribution
* Canonical Distribution of Kubernetes

# TL;DR;

**Disclaimer:** This does not fully work, yet xD...
Multiple scenarios might be broken, the DNS might not work properly
and the HAProxy service might be failing also, this is the reason why is not documented
in the [official docs](https://docs.kubeinit.com). Any contribution is and always be
welcomed. Yet, the deployment should finish successfully.

### The roles and playbooks structure

Every supported distro has a role folder called like `kubeinit_<distro>`, this means,
`kubeinit_okd`, `kubeinit_k8s`, `kubeinit_rke`, and `kubeinit_cdk`.

Then, there is a specific playbook for each distro named using their distribution
initials like `okd`, `k8s`, `rke`, and `cdk`.

This means that the workflow to deploy any distro must be the same for every one of them.

### Deploying

Choose the Kubernetes distribution you will use:

* Origin Kubernetes Distribution: okd
* Kubernetes: k8s
* Rancher Kubernetes Distribution: rke
* Canonical Distribution of Kubernetes : cdk

```bash
# Choose the distro
# distro=k8s
# distro=rke
# distro=cdk
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

Being able to deploy multiple Kubernetes distributions in an easy, quick,
reproducible, and using the same interface allow users to test and evaluate
them to see which one (or many) fits better their use cases.

### The end

If you like this post, please try the code, raise issues, and ask for more details, features or
anything that you feel interested in. Also it would be awesome if you become a stargazer to catch up
updates and new features.

This is the main project [repository](https://github.com/kubeinit/kubeinit).

Happy KubeIniting!

---

![](/static/kubeinit/yaml.jpeg)
