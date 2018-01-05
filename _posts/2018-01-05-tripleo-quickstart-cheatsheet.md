---
layout: post
title: "New TripleO quickstart cheatsheet"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
favorite: true
commentIssueId: 42
---

I have created some cheatsheets for people starting to work on TripleO,
mostly to help them to bootstrap a development environment as soon as possible.

[The previous version](https://github.com/ccamacho/tripleo-graphics/tree/master/cheatsheets/old_style)
of this cheatsheet series was used in
several community conferences (FOSDEM, DevConf.cz),
now, they are deprecated as
they way TripleO should be deployed changed considerably last months.

Here you have the last version:

![](/static/01-tripleo-cheatsheet-deploying-tripleo_p1.jpg)

![](/static/01-tripleo-cheatsheet-deploying-tripleo_p2.jpg)

The source code of these bookmarks is available as usual on
[GitHub](https://github.com/ccamacho/tripleo-graphics/tree/master/cheatsheets/latest_style)

And this is the code if you want to execute it directly:

```
# 01 - Create the toor user.
sudo useradd toor
echo "toor:toor" | chpasswd
echo "toor ALL=(root) NOPASSWD:ALL" \
  | sudo tee -a /etc/sudoers.d/toor
sudo chmod 0440 /etc/sudoers.d/toor
su - toor

# 02 - Prepare the hypervisor node.
cd
mkdir .ssh
ssh-keygen -t rsa -N "" -f .ssh/id_rsa
cat .ssh/id_rsa.pub >> .ssh/authorized_keys
sudo bash -c "cat .ssh/id_rsa.pub \
  >> /root/.ssh/authorized_keys"
sudo bash -c "echo '127.0.0.1 127.0.0.2' \
  >> /etc/hosts"
export VIRTHOST=127.0.0.2
sudo yum groupinstall "Virtualization Host" -y
sudo yum install git lvm2 lvm2-devel -y
ssh root@$VIRTHOST uname -a

# 03 - Clone repos and install deps.
git clone \
  https://github.com/openstack/tripleo-quickstart
chmod u+x ./tripleo-quickstart/quickstart.sh
bash ./tripleo-quickstart/quickstart.sh \
  --install-deps
sudo setenforce 0

# 04 - Configure the TripleO deployment with Docker and HA.
export CONFIG=~/deploy-config.yaml
cat > $CONFIG << EOF
overcloud_nodes:
  - name: control_0
    flavor: control
    virtualbmc_port: 6230
  - name: compute_0
    flavor: compute
    virtualbmc_port: 6231
node_count: 2
containerized_overcloud: true
delete_docker_cache: true
enable_pacemaker: true
run_tempest: false
extra_args: >-
  --libvirt-type qemu
  --ntp-server pool.ntp.org
  -e /usr/share/openstack-tripleo-heat-templates/environments/docker.yaml
  -e /usr/share/openstack-tripleo-heat-templates/environments/docker-ha.yaml
EOF

# 05 - Deploy TripleO.
export VIRTHOST=127.0.0.2
bash ./tripleo-quickstart/quickstart.sh \
      --clean          \
      --release master \
      --teardown all   \
      --tags all       \
      -e @$CONFIG      \
      $VIRTHOST
```

Happy TripleOing!!!
