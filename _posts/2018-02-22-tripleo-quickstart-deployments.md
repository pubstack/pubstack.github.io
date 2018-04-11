---
layout: post
title: "Deployment configurations"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
  - draft
#favorite: true
#commentIssueId: 42
---



This post is a summary of the deployments I ussualy test.
The ussual steps are:

__01 - Create the toor user.__

```
sudo useradd toor
echo "toor:toor" | sudo chpasswd
echo "toor ALL=(root) NOPASSWD:ALL" \
  | sudo tee -a /etc/sudoers.d/toor
sudo chmod 0440 /etc/sudoers.d/toor
su - toor
```

__02 - Prepare the hypervisor node.__


```
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
```

__03 - Clone repos and install deps.__


```
git clone \
  https://github.com/openstack/tripleo-quickstart
chmod u+x ./tripleo-quickstart/quickstart.sh
bash ./tripleo-quickstart/quickstart.sh \
  --install-deps
sudo setenforce 0
```

__04 - Export common variables.__

```
export CONFIG=~/deploy-config.yaml
export VIRTHOST=127.0.0.2
```

__05 - Click on the environment description to expand the recipe.__


<details>
<summary><strong>[Containerized & HA] - 1 Controller 1, Compute</strong></summary>
<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
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
</code></pre></div></div>
</details>

<details>
<summary><strong>[Containerized & HA] - 3 Controllers, 1 Compute</strong></summary>
<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
cat > $CONFIG << EOF
overcloud_nodes:
  - name: control_0
    flavor: control
    virtualbmc_port: 6230
  - name: control_1
    flavor: control
    virtualbmc_port: 6231
  - name: control_2
    flavor: control
    virtualbmc_port: 6232
  - name: compute_1
    flavor: compute
    virtualbmc_port: 6233
node_count: 4
containerized_overcloud: true
delete_docker_cache: true
enable_pacemaker: true
run_tempest: false
extra_args: >-
  --libvirt-type qemu
  --ntp-server pool.ntp.org
  --control-scale 3
  --compute-scale 1
  -e /usr/share/openstack-tripleo-heat-templates/environments/docker.yaml
  -e /usr/share/openstack-tripleo-heat-templates/environments/docker-ha.yaml
EOF
</code></pre></div></div>
</details>
<br/>



__06 - Deploy TripleO.__

```
bash ./tripleo-quickstart/quickstart.sh \
      --clean          \
      --release master \
      --teardown all   \
      --tags all       \
      -e @$CONFIG      \
      $VIRTHOST
```
