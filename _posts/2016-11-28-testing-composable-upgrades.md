---
layout: post
title: "Testing composable upgrades"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
  - cloud
commentIssueId: 25
refimage: '/static/tripleo_banner.png'
---

This is a brief recipe about how I'm testing
composable upgrades O->P.

Based on the original shardy's notes
from [this](http://paste.openstack.org/show/590436/) link.

The following steps are followed to upgrade your Overcloud from Ocata to latest master (Pike).

- Deploy latest master TripleO following [this](http://www.pubstack.com/blog/2016/07/04/manually-installing-tripleo-recipe.html) post.

- Remove the current Overcloud deployment.

```
  source stackrc
  heat stack-delete overcloud
```

- Remove the Overcloud images and create new ones (for the  Overcloud).

```
  cd
  openstack image list
  openstack image delete <image_ID> #Delete all the Overcloud images overcloud-full*
  rm -rf /home/stack/overcloud-full.*

  export STABLE_RELEASE=ocata
  export USE_DELOREAN_TRUNK=1
  export DELOREAN_TRUNK_REPO="https://trunk.rdoproject.org/centos7-ocata/current/"
  export DELOREAN_REPO_FILE="delorean.repo"
  /home/stack/tripleo-ci/scripts/tripleo.sh --overcloud-images

  # Or reuse images
  # wget https://images.rdoproject.org/ocata/delorean/current-tripleo/stable/overcloud-full.tar
  # tar -xvf overcloud-full.tar
  # openstack overcloud image upload --update-existing
```

- Download Ocata tripleo-heat-templates.

```
  cd
  git clone -b stable/ocata https://github.com/openstack/tripleo-heat-templates tht-ocata
```

- Configure the DNS (needed when upgrading the Overcloud).

```
  neutron subnet-update `neutron subnet-list | grep ctlplane-subnet | awk '{print $2}'` --dns-nameserver 192.168.122.1
```

- Deploy an Ocata Overcloud.

```
  openstack overcloud deploy \
  --libvirt-type qemu \
  --ntp-server pool.ntp.org \
  --templates /home/stack/tht-ocata/ \
  -e /home/stack/tht-ocata/overcloud-resource-registry-puppet.yaml \
  -e /home/stack/tht-ocata/environments/puppet-pacemaker.yaml
```

- Install prerequisites in nodes (if no DNS configured this will fail, so make sure they have Intenet access), check that your nodes can connect to Internet.

```
cat > upgrade_repos.yaml << EOF
parameter_defaults:
  UpgradeInitCommand: |
    set -e

    #Master repositories
    sudo curl -o /etc/yum.repos.d/delorean.repo https://trunk.rdoproject.org/centos7-master/current-passed-ci/delorean.repo
    sudo curl -o /etc/yum.repos.d/delorean-deps.repo https://trunk.rdoproject.org/centos7/delorean-deps.repo


    export HOME=/root
    cd /root/
    if [ ! -d tripleo-ci ]; then
      git clone https://github.com/openstack-infra/tripleo-ci.git
    else
      pushd tripleo-ci
      git checkout master
      git pull
      popd
    fi

    if [ ! -d tripleo-heat-templates ]; then
      git clone https://github.com/openstack/tripleo-heat-templates.git
    else
      pushd tripleo-heat-templates
      git checkout master
      git pull
      popd
    fi

    ./tripleo-ci/scripts/tripleo.sh --repo-setup
    sed -i "s/includepkgs=/includepkgs=python-heat-agent*,/" /etc/yum.repos.d/delorean-current.repo
    #yum -y install python-heat-agent-ansible
    yum install -y python-heat-agent-*

    rm -f /usr/libexec/os-apply-config/templates/etc/puppet/hiera.yaml
    rm -f /usr/libexec/os-refresh-config/configure.d/40-hiera-datafiles
    rm -f /etc/puppet/hieradata/*.yaml
    yum remove -y python-UcsSdk openstack-neutron-bigswitch-agent python-networking-bigswitch openstack-neutron-bigswitch-lldp python-networking-odl
    crudini --set /etc/ansible/ansible.cfg DEFAULT library /usr/share/ansible-modules/
EOF
```

- Download master tripleo-heat-templates.

```
  cd
  git clone https://github.com/openstack/tripleo-heat-templates tht-master
```

- Upgrade Overcloud to master

```
  cd
  openstack overcloud deploy \
  --libvirt-type qemu \
  --ntp-server pool.ntp.org \
  --templates /home/stack/tht-master/ \
  -e /home/stack/tht-master/overcloud-resource-registry-puppet.yaml \
  -e /home/stack/tht-master/environments/puppet-pacemaker.yaml \
  -e /home/stack/tht-master/environments/major-upgrade-composable-steps.yaml \
  -e upgrade_repos.yaml
```

Note: if upgrading to a containerized Overcloud (Pike and beyond) do:

```
cat > docker_registry.yaml << EOF
parameter_defaults:
  DockerNamespace: 192.168.24.1:8787/tripleoupstream
  DockerNamespaceIsRegistry: true
EOF

# This will take some time...
openstack overcloud container image upload --config-file /usr/share/openstack-tripleo-common/container-images/overcloud_containers.yaml

openstack overcloud container image prepare \
--namespace tripleoupstream \
--tag latest \
--env-file docker-centos-tripleoupstream.yaml

cd
source ~/stackrc
export THT=/home/stack/tht-master

openstack overcloud deploy --templates $THT \
--libvirt-type qemu \
--ntp-server pool.ntp.org \
-e $THT/overcloud-resource-registry-puppet.yaml \
-e $THT/environments/puppet-pacemaker.yaml \
-e $THT/environments/major-upgrade-composable-steps.yaml \
-e upgrade_repos.yaml \
-e $THT/environments/docker.yaml \
-e $THT/environments/docker-ha.yaml \
-e $THT/environments/major-upgrade-composable-steps-docker.yaml \
-e docker-centos-tripleoupstream.yaml \
-e docker_registry.yaml
```

- Run the converge step ** Not tested on the containerized upgrade **

```
  cd
  openstack overcloud deploy \
  --libvirt-type qemu \
  --ntp-server pool.ntp.org \
  --templates /home/stack/tht-master/ \
  -e /home/stack/tht-master/overcloud-resource-registry-puppet.yaml \
  -e /home/stack/tht-master/environments/puppet-pacemaker.yaml \
  -e /home/stack/tht-master/environments/major-upgrade-converge.yaml
```

If the last steps manage to finish successfully, you just have upgraded your Overcloud from Ocata to Pike (latest master).

For more resources related to TripleO deployments, check out the [TripleO YouTube channel](https://www.youtube.com/channel/UCNGDxZGwUELpgaBoLvABsTA).

<div style="font-size:10px">
  <blockquote>
    <p><strong>Updated 2017/01/28:</strong> Working fine.</p>
  </blockquote>
</div>
