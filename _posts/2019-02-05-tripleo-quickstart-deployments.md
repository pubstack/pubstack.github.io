---
layout: post
title: "TripleO - Deployment configurations"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
  - cloud
favorite: true
commentIssueId: 57
refimage: '/static/dude-just-deploy-it-already.jpg'
---

This post is a summary of the deployments I usually test for deploying TripleO
using quickstart.


![](/static/dude-just-deploy-it-already.jpg)


The following steps need to run in the Hypervisor node
in order to deploy both the Undercloud and the Overcloud.

You need to execute them one after the other, the idea of this recipe is to
have something just for copying/pasting.

Once the last step ends you can/should be able to  connect to the
Undercloud VM to start operating your Overcloud deployment.

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

# Disable IPv6 lookups
# sudo bash -c "cat >> /etc/sysctl.conf" << EOL
# net.ipv6.conf.all.disable_ipv6 = 1
# net.ipv6.conf.default.disable_ipv6 = 1
# EOL
# sudo sysctl -p

# Enable IPv6 in kernel cmdline
# sed -i s/ipv6.disable=1/ipv6.disable=0/ /etc/default/grub
# grub2-mkconfig -o /boot/grub2/grub.cfg
# reboot

sudo yum groupinstall "Virtualization Host" -y
sudo yum install git lvm2 lvm2-devel -y
sudo yum install libvirt-python python-lxml libvirt -y
```


__02 - Create the toor user (from the Hypervisor node, as root).__

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
cat .ssh/id_rsa.pub >> .ssh/authorized_keys
cat .ssh/id_rsa.pub | sudo tee -a /root/.ssh/authorized_keys
echo '127.0.0.1 127.0.0.2' | sudo tee -a /etc/hosts

export VIRTHOST=127.0.0.2
ssh root@$VIRTHOST uname -a
```

Now, follow as the `toor` user and prepare the Hypervisor node
for the deployment.

__03 - Clone repos and install deps.__


```bash
git clone \
  https://github.com/openstack/tripleo-quickstart
chmod u+x ./tripleo-quickstart/quickstart.sh
bash ./tripleo-quickstart/quickstart.sh \
  --install-deps
sudo setenforce 0
```

Export some variables used in the deployment command.

__04 - Export common variables.__

```bash
export CONFIG=~/deploy-config.yaml
export VIRTHOST=127.0.0.2
```

Now we will create the configuration file used for the deployment,
depending on the file you choose you will deploy different environments.

__05 - Click on the environment description to expand the recipe.__


<details>
<summary><strong>OpenStack [Containerized & HA] - 1 Controller, 1 Compute</strong></summary>
<pre><code class="language-bash">
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
  -e /usr/share/openstack-tripleo-heat-templates/environments/docker-ha.yaml
EOF
</code></pre>
</details>

<details>
<summary><strong>OpenStack [Containerized & HA] - 3 Controllers, 1 Compute</strong></summary>
<pre><code class="language-bash">
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
  -e /usr/share/openstack-tripleo-heat-templates/environments/docker-ha.yaml
EOF
</code></pre>
</details>

<details>
<summary><strong>OpenShift [Containerized] - 1 Controller, 1 Compute</strong></summary>
<pre><code class="language-bash">
cat > $CONFIG << EOF
# Original from https://github.com/openstack/tripleo-quickstart/blob/master/config/general_config/featureset033.yml
composable_scenario: scenario009-multinode.yaml
deployed_server: true

network_isolation: false
enable_pacemaker: false
overcloud_ipv6: false
containerized_undercloud: true
containerized_overcloud: true

# This enables TLS for the undercloud which will also make haproxy bind to the
# configured public-vip and admin-vip.
undercloud_generate_service_certificate: false
undercloud_enable_validations: false

# This enables the deployment of the overcloud with SSL.
ssl_overcloud: false

# Centos Virt-SIG repo for atomic package
add_repos:
  # NOTE(trown) The atomic package from centos-extras does not work for
  # us but its version is higher than the one from the virt-sig. Hence,
  # using priorities to ensure we get the virt-sig package.
  - type: package
    pkg_name: yum-plugin-priorities
  - type: generic
    reponame: quickstart-centos-paas
    filename: quickstart-centos-paas.repo
    baseurl: https://cbs.centos.org/repos/paas7-openshift-origin311-candidate/x86_64/os/
  - type: generic
    reponame: quickstart-centos-virt-container
    filename: quickstart-centos-virt-container.repo
    baseurl: https://cbs.centos.org/repos/virt7-container-common-candidate/x86_64/os/
    includepkgs:
      - atomic
    priority: 1

extra_args: ''

container_args: >-
  # If Pike or Queens
  #-e /usr/share/openstack-tripleo-heat-templates/environments/docker.yaml
  # If Ocata, Pike, Queens or Rocky
  #-e /home/stack/containers-default-parameters.yaml
  # If >= Stein
  -e /home/stack/containers-prepare-parameter.yaml

  -e /usr/share/openstack-tripleo-heat-templates/openshift.yaml
# NOTE(mandre) use container images mirrored on the dockerhub to take advantage
# of the proxy setup by openstack infra
docker_openshift_etcd_namespace: docker.io/{{ docker_registry_namespace }}
docker_openshift_cluster_monitoring_namespace: docker.io/tripleomaster
docker_openshift_cluster_monitoring_image: coreos-cluster-monitoring-operator
docker_openshift_configmap_reload_namespace: docker.io/tripleomaster
docker_openshift_configmap_reload_image: coreos-configmap-reload
docker_openshift_prometheus_operator_namespace: docker.io/tripleomaster
docker_openshift_prometheus_operator_image: coreos-prometheus-operator
docker_openshift_prometheus_config_reload_namespace: docker.io/tripleomaster
docker_openshift_prometheus_config_reload_image: coreos-prometheus-config-reloader
docker_openshift_kube_rbac_proxy_namespace: docker.io/tripleomaster
docker_openshift_kube_rbac_proxy_image: coreos-kube-rbac-proxy
docker_openshift_kube_state_metrics_namespace: docker.io/tripleomaster
docker_openshift_kube_state_metrics_image: coreos-kube-state-metrics

deploy_steps_ansible_workflow: true
config_download_args: >-
  -e /home/stack/config-download.yaml
  --disable-validations
  --verbose
composable_roles: true

overcloud_roles:
  - name: Controller
    CountDefault: 1
    tags:
      - primary
      - controller
    networks:
      - External
      - InternalApi
      - Storage
      - StorageMgmt
      - Tenant
  - name: Compute
    CountDefault: 0
    tags:
      - compute
    networks:
      - External
      - InternalApi
      - Storage
      - StorageMgmt
      - Tenant

tempest_config: false
test_ping: false
run_tempest: false
EOF
</code></pre>
</details>
<br/>

From the Hypervisor, as the `toor` user
run the deployment command to deploy
both your Undercloud and Overcloud.

__06 - Deploy TripleO.__

```bash
bash ./tripleo-quickstart/quickstart.sh \
      --clean          \
      --release master \
      --teardown all   \
      --tags all       \
      -e @$CONFIG      \
      $VIRTHOST
```

<div style="font-size:10px">
  <blockquote>
    <p><strong>Updated 2019/02/05:</strong> Initial version.</p>
    <p><strong>Updated 2019/02/05:</strong> TODO: Test the OpenShift deployment.</p>
    <p><strong>Updated 2019/02/06:</strong> Added some clarifications about where the commands should run.</p>
  </blockquote>
</div>
