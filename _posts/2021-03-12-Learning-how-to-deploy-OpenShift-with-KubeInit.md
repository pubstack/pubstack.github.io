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

Then the next step is to fetch a valid registry tokens list (pullsecrets) from
[https://cloud.redhat.com/openshift/install/pull-secret](https://cloud.redhat.com/openshift/install/pull-secret).

You should get a very long json object as a dictionary with the credential details
we need to adjust our deployment pull secrets to be able to "fetch"
the images accordingly.

The pullsecret syntax should look like:

```bash
{
  "auths":{
    "cloud.openshift.com":{"auth":"TOKEN1_GOES_HERE","email":"email@example"},
    "quay.io":{"auth":"TOKEN2_GOES_HERE","email":"email@example.com"},
    "registry.connect.redhat.com":{"auth":"TOKEN3_GOES_HERE","email":"email@example.com"},
    "registry.redhat.io":{"auth":"TOKEN4_GOES_HERE","email":"email@example.com"}
  }
}
```

### Deploying

The deployment procedure is the same
as it is for all the other Kubernetes distributions that can be
deployed with KubeInit.

Please follow the [usage documentation](http://docs.kubeinit.com/usage.html)
to understand the system's requirements and the required host supported
Linux distributions.

At the moment we will deploy OpenShift 4.7.0 (the latest release available),
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
    -e kubeinit_okd_openshift_deploy=True \
    -e kubeinit_okd_openshift_registry_token_cloud_openshift_com="TOKEN1_GOES_HERE" \
    -e kubeinit_okd_openshift_registry_token_quay_io="TOKEN2_GOES_HERE" \
    -e kubeinit_okd_openshift_registry_token_registry_connect_redhat_com="TOKEN3_GOES_HERE" \
    -e kubeinit_okd_openshift_registry_token_registry_redhat_io="TOKEN4_GOES_HERE" \
    -e kubeinit_okd_openshift_registry_token_email="email@example.com" \
    ./playbooks/$distro.yml

```

Note: The variables required to override an
OpenShift deployment are
`kubeinit_okd_openshift_deploy`,
`kubeinit_okd_openshift_registry_token_cloud_openshift_com`,
`kubeinit_okd_openshift_registry_token_quay_io`,
`kubeinit_okd_openshift_registry_token_registry_connect_redhat_com`,
`kubeinit_okd_openshift_registry_token_registry_redhat_io`, and
`kubeinit_okd_openshift_registry_token_email`.


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
