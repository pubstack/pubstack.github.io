---
layout: post
title: "Learning how to deploy OpenShift with KubeInit"
author: "Carlos Camacho"
categories:
  - blog
tags:
- kubernetes
- kubeinit
- openshift
favorite: false
commentIssueId: 81
refimage: '/static/kubeinit/ki_ocp.png'
---

KubeInit is an Ansible collection to ease
the deployment of multiple Kubernetes distributions.
This post will show you how to use it to deploy
OpenShift in your infrastructure.

# TL;DR;

This post will show you the command and the parameters
that needs to be configured to deploy OpenShift (4.7).

### Prerequirements

Adjust your inventory file accordingly to what you will like to deploy.
Please, make sure you read older posts to understand KubeInit's deployment
workflow, or give the docs a try
at [https://docs.kubeinit.com/](https://docs.kubeinit.com/).

### OpenShift registry token

Then the next step is to fetch a valid registry token from
[https://cloud.redhat.com/](https://cloud.redhat.com/)
in particular from
[https://cloud.redhat.com/openshift/token](https://cloud.redhat.com/openshift/token).

You should get a very long string like "eyJhbGci.........CIgOiAiSldUIiwia2lkIiA6I",
having the string we need to adjust our deployment pull secret to be able to "fetch"
the images like `'  {"auths":{"cloud.openshift.com":{"auth":<the token goes here>}}}'`
be aware of the two leading spaces.

The spaces after the first single quote are required,
do not remove them as Ansible appears to be recognizing this as valid Python list,
so it's getting transformed into a Python list and then serialized
using Python's str(), which is why we end up with the single-quoted values.

### Deploying

The deployment procedure is the same
as it is for all the other Kubernetes distributions that can be
deployed with KubeInit.

Please follow the [usage documentation](http://docs.kubeinit.com/usage.html)
to understand the system's requirements and the required host supported
Linux distributions.

At the moment we will deploy OpenShift 4.7.1 (the latest release available),
if you need to deploy other releases adjust the value of the
`kubeinit_okd_registry_release_tag` variable.

```bash
# Choose the distro
distro=okd
# Run the deployment command
ansible-playbook \
    -v \
    --user root \
    -i ./hosts/$distro/inventory \
    --become \
    --become-user root \
    -e "{ \
      'kubeinit_okd_openshift_deploy': 'true', \
      'kubeinit_okd_registry_pullsecret': '  {\"auths\":{\"cloud.openshift.com\":{\"auth\": \"eyJh...2gX3TgXk\"}}}', \
    }" \
    ./playbooks/$distro.yml

```

Note: The only two variables we updated were to override that
we will like to deploy OpenShift and our registry token.

Note: The auth key must be enclosed in double quotes escaped with
backslashes.

### Conclusions

Deploying also OpenShift demostrates how
flexible KubeInit can be.
With a few changes we can deploy also downstream Kubernetes
distributions ready to be used for production grade deployments.

Once [#219](https://github.com/Kubeinit/kubeinit/pull/219/files)
is merged you should be able to run this.

### The end

If you like this post, please try the code, raise issues, and ask for more details, features or
anything that you feel interested in. Also it would be awesome if you become a stargazer to catch up
updates and new features.

This is the main project [repository](https://github.com/kubeinit/kubeinit).

Happy KubeIniting!
