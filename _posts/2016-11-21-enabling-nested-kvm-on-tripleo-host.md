---
layout: post
title: "Enabling nested KVM support for a instack-virt-setup deployment."
author: "Carlos Camacho"
date: 2016-11-21 12:30:00
categories:
  - blog
tags:
  - tripleo
  - openstack
  - cloud
commentIssueId: 19
refimage: '/static/tripleo_banner.png'
---

The following bash snippet will enable
nested KVM support in the host when deploying TripleO
using instack-virt-setup.

This will work in AMD or Intel architectures.

```bash
#!/bin/bash
echo "Checking if nested KVM is enabled in the host."
ARCH=$(lscpu | grep Architecture | head -1 | awk '{print $2}')
if [[ $ARCH == 'x86_64' ]]; then
    ARCH_BRAND=intel
    KVM_STATUS_FILE=/sys/module/kvm_intel/parameters/nested
    ENABLE_NESTED_KVM=Y
else
    ARCH_BRAND=amd
    KVM_STATUS_FILE=/sys/module/kvm_amd/parameters/nested
    ENABLE_NESTED_KVM=1
fi
if [[ -f $KVM_STATUS_FILE ]]; then
    KVM_CURRENT_STATUS=$(head -n 1 $KVM_STATUS_FILE)
    if [[ "${KVM_CURRENT_STATUS^^}" -ne "${ENABLE_NESTED_KVM^^}" ]]; then
        echo "This host does not have nested KVM enabled, enabling."
        sudo rmmod kvm-$ARCH_BRAND
        sudo sh -c "echo 'options kvm-$ARCH_BRAND nested=$ENABLE_NESTED_KVM' >> /etc/modprobe.d/dist.conf"
        sudo modprobe kvm-$ARCH_BRAND
    else
        echo "Nested KVM support is already enabled."
    fi
else
    echo "$KVM_STATUS_FILE does not exist."
fi
```

By default nested virtualization with KVM is disabled in the
host, so in order to run the overcloud-pingtest correctly we have two
options. Either run the previous snippet on the host,
or, when deploying the Compute node in a virtual machine
add `--libvirt-type qemu` to the deployment command.
Otherwise launching instances on the deployed overcloud will fail.

Here you have an example of the deployment command, fixing
libvirt to qemu.

```bash
cd
openstack overcloud deploy \
--libvirt-type qemu \
--ntp-server pool.ntp.org \
--templates /home/stack/tripleo-heat-templates \
-e /home/stack/tripleo-heat-templates/overcloud-resource-registry-puppet.yaml \
-e /home/stack/tripleo-heat-templates/environments/puppet-pacemaker.yaml
```

Have a happy TripleO deployment!
