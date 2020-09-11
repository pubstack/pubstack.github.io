---
layout: post
title: "Deploying KubeInit from a container"
author: "Carlos Camacho"
categories:
  - blog
tags:
- kubernetes
- kubeinit
- cloud
- okd
favorite: true
commentIssueId: 71
refimage: '/static/kubeinit/kubeinit_in_docker.png'
---

There are some use cases where users have old libraries versions,
old environments, or in general, difficulties to execute the
ansible-playbook command to deploy KubeInit, due to unrelated
issues. The following steps will help users to deploy KubeInit by
launching the ansible-playbook command from a container.

![](/static/kubeinit/kubeinit_in_docker.png)

# TL;DR;

We will describe how to run KubeInit within a container.


### Requirements

* Having docker or podman installed.
* If you do not have podman (what is used in the commands bellow),
  then replace podman by docker in all the commands bellow.

### Deploying KubeInit within a container

#### Building the image

We will clone the repository as usual, and build the container image.

```bash
git clone https://github.com/Kubeinit/kubeinit.git
cd kubeinit
podman build -t kubeinit/kubeinit .
```

#### Run the deployment command from the recently created container

**_NOTE:_** Beware of the [:z flag if you have SELinux enabled](https://www.redhat.com/sysadmin/user-namespaces-selinux-rootless-containers)
you must use it, otherwise you will get a permissions denied as the PK won't be able to be mounted correctly.

**_NOTE:_** Each time a change is included in any code inside
the repository **BUILD THE IMAGE YOU MUST**...

```bash
podman run --rm -it \
    -v ~/.ssh/id_rsa:/root/.ssh/id_rsa:z \
    -v /etc/hosts:/etc/hosts \
    kubeinit/kubeinit \
        --user root \
        -v -i ./hosts/okd/inventory \
        --become \
        --become-user root \
        ./playbooks/okd.yml
```
As it is clearly visible in the previous command, starting from
the 5th line, the deployment command is exactly as if we were
executing it with `ansible-playbook`.

What we are doing is to add `ansible-playbook` as this container
ENTRYPOINT, so you can add any variable that will be part
of the `ansible-playbook` command.

Easy as always and in a single deployment command
you should have your cluster in approximately 30 minutes.

### Pros and Cons of executing ansible-playbook within a container

This is a very opinionated section to show that sometimes running
containers can add some more extra steps that might be useful or not
depending on your environment.

#### Pros

* No dependencies from the host.
* Easy to run if there is no other change to make inside the collection.

#### Cons

* Debugging will be always harder.
* Another layer of something that might be hard to understand.
* More time to build the image and deploying.
* Each time you need to make a change in any part of the code you will need to build the image again.
  This in particular can be really painful, as for each small change in the code it will make
  you invest some maybe unneeded extra time to be able to run the code again.

#### The end

If you like this post, please try the code, raise issues, and ask for more details, features or
anything that you feel interested in. Also it would be awesome if you become a stargazer to catch up
updates and new features.

This is the main project [repository](https://github.com/kubeinit/kubeinit).

Happy KubeIniting!

---

![](/static/pod/k8s.jpg)
