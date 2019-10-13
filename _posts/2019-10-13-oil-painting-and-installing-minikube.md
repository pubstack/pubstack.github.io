---
layout: post
title: "Oil painting and Minikube - Install your development environment with Minikube"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - kubernetes
  - minikube
  - cloud
commentIssueId: 64
refimage: '/static/minikube.jpeg'
---

Today I got some time to do some oil painting and reading about techy stuff :)

This post is a brief summary of the
deployment steps for installing Minikube in a Centos 7
baremetal machine, and, to show you my painting (check the fedora!).

![](/static/Terraza-En-Grecia-by-Carlos-Camacho.jpg)


The following steps need to run in the Hypervisor machine
in which you will like to have your Minikube deployment.

You need to execute them one after the other,
the idea of this recipe is to
have something just for copying/pasting.


The usual steps are:

__01 - Prepare the hypervisor node.__

Now, let's install some dependencies.
Same Hypervisor node, same `root` user.

```bash
# In this dev. env. /var is only 50GB, so I will create
# a sym link to another location with more capacity.
# It will take easily more tan 50GB deploying a 3+1 overcloud
sudo mkdir -p /home/libvirt/
sudo ln -sf /home/libvirt/ /var/lib/libvirt

# Install some packages ***Including EPEL***
sudo yum install epel-release -y
sudo yum groupinstall "Virtualization Host" -y
sudo yum install libvirt qemu-kvm virt-install virt-top libguestfs-tools bridge-utils -y
sudo yum install git lvm2 lvm2-devel -y
sudo yum install libvirt-python python-lxml libvirt curl-y
sudo yum install binutils qt gcc make patch libgomp -y
sudo yum install glibc-headers glibc-devel kernel-headers -y
sudo yum install kernel-devel dkms bash-completion -y
sudo yum install nano wget -y
sudo yum install python3-pip -y 
```

__02 - Check that the kernel modules are OK.__

```bash
# Check the kernel modules are OK
sudo lsmod | grep kvm
```

__03 - Enable libvirtd, disable SElinux xD and firewalld.__

```bash
# Enable libvirtd
sudo systemctl start libvirtd
sudo systemctl enable libvirtd

# Disable selinux & stop firewall as needed.
setenforce 0
perl -pi -e 's/SELINUX\=enforcing/SELINUX\=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

__04 - Install Minikube.__

```bash
#Install minikube
/usr/bin/curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
cp -p minikube /usr/local/bin && rm -f minikube

# Create the repo for kubernetes
cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# Install kubectl
sudo yum install kubectl -y
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

__05 - Create the toor user (from the Hypervisor node, as root).__

```bash
sudo useradd toor
echo "toor:toor" | sudo chpasswd
echo "toor ALL=(root) NOPASSWD:ALL" \
  | sudo tee /etc/sudoers.d/toor
sudo chmod 0440 /etc/sudoers.d/toor
sudo su - toor

cd
mkdir .ssh
ssh-keygen -t rsa -N "" -f .ssh/id_rsa
```

Now, follow as the `toor` user and prepare the Hypervisor node
for Minikube.

__06 - Finish the Minikube configuration.__

```bash
# Add to bashrc in toor user
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

# We add toor to the libvirtd group
sudo usermod --append --groups libvirt toor
```

__07 - Start Minikube.__

```bash
minikube start --memory=65536 --cpus=4 --vm-driver kvm2
export no_proxy=$no_proxy,$(minikube ip)
nohup kubectl proxy --address='0.0.0.0' --port=8001 --disable-filter=true &
sleep 30
minikube addons enable dashboard
nohup minikube dashboard &
minikube addons open dashboard
```

The Minikube instance should be reachable from the following URL:

http://machine_ip:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ 

```bash
# To stop/delete
kubectl delete deploy,svc --all
minikube stop
minikube delete
```

__08 - Minikube cheat sheet.__

```bash
# set & get current context of cluster
 kubectl config use-context minikube
 kubectl config current-context

# fetch all the kubernetes objects for a namespace
 kubectl get all -n kube-system

# display cluster details
 kubectl cluster-info

# set custom memory and cpu 
 minikube config set memory 4096
 minikube config set cpus 2

# fetch cluster ip
 minikube ip

# ssh to the minikube vm
 minikube ssh

# display addons list and status
 minikube addons list

# exposes service to vm & retrieves url 
 minikube service elasticsearch
 minikube service elasticsearch --url
```

<div style="font-size:10px">
  <blockquote>
    <p><strong>Updated 2019/10/13:</strong> Initial version.</p>
  </blockquote>
</div>
