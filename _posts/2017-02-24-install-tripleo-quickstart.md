---
layout: post
title: "Installing TripleO Quickstart"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
favorite: true
commentIssueId: 29
---

This is a brief recipe about how to
manually install TripleO Quickstart in a remote
32GB RAM box and not dying trying it.

Now `instack-virt-setup` is deprecated :( :( :(
so the manual process needs to evolve and use OOOQ (TripleO Quickstart).

This post is a brief recipe about how to provision the Hypervisor node
and deploy an end-to-end development environment
based on TripleO-Quickstart.

From the hypervisor run:

```bash
# In this dev. env. /var is only 50GB, so I will create
# a sym link to another location with more capacity.
# It will take easily more tan 50GB deploying a 3+1 overcloud
sudo mkdir -p /home/libvirt/
sudo ln -sf /home/libvirt/ /var/lib/libvirt
# Add default toor user
sudo useradd toor
echo "toor:toor" | chpasswd
echo "toor ALL=(root) NOPASSWD:ALL" | sudo tee -a /etc/sudoers.d/toor
sudo chmod 0440 /etc/sudoers.d/toor
su - toor
whoami

sudo yum groupinstall "Virtualization Host" -y
sudo yum install git -y

cd
mkdir .ssh
ssh-keygen -t rsa -N "" -f .ssh/id_rsa
cat .ssh/id_rsa.pub >> .ssh/authorized_keys
sudo bash -c "cat .ssh/id_rsa.pub >> /root/.ssh/authorized_keys"
sudo bash -c "echo '127.0.0.1 127.0.0.2' >> /etc/hosts"
export VIRTHOST=127.0.0.2

ssh root@$VIRTHOST uname -a
```

Now, we can start deploying TripleO Quickstart by following:

```bash
# Source: http://rdo-ci-doc.usersys.redhat.com/docs/tripleo-environments/en/latest/oooq-downstream.html
# Downstream bits for OSP8 ...
# cd
# sudo yum -y install /usr/bin/c_rehash ca-certificates
# sudo update-ca-trust check
# sudo update-ca-trust force-enable
# sudo update-ca-trust enable
# wget cert.pem
# sudo cp cert.pem /etc/pki/tls/certs/
# sudo cp cert.pem /etc/pki/ca-trust/source/anchors/
# sudo c_rehash
# sudo update-ca-trust extract
# git clone https://github.com/openstack/tripleo-quickstart
# cd tripleo-quickstart
# wget http://rhos-release.virt.bos.redhat.com/ci-images/internal-requirements-new.txt
# cd
# chmod u+x ./tripleo-quickstart/quickstart.sh
# bash ./tripleo-quickstart/quickstart.sh --install-deps
# bash ./tripleo-quickstart/quickstart.sh -v --release rhos-8-baseos-undercloud --clean --teardown all --requirements "/home/toor/tripleo-quickstart/internal-requirements-new.txt" $VIRTHOST

```

```bash
git clone https://github.com/openstack/tripleo-quickstart
chmod u+x ./tripleo-quickstart/quickstart.sh
printf "\n\nSee:\n./tripleo-quickstart/quickstart.sh --help for a full list of options\n\n"
bash ./tripleo-quickstart/quickstart.sh --install-deps

export VIRTHOST=127.0.0.2
export CONFIG=~/deploy-config.yaml

cat > $CONFIG << EOF

# undercloud_undercloud_hostname: undercloud.ratata-domain

control_memory: 8192
compute_memory: 6120
 
undercloud_memory: 10240
undercloud_vcpu: 4
undercloud_workers: 3

default_vcpu: 1

custom_nameserver: '10.16.36.29'
undercloud_undercloud_nameservers: '10.16.36.29'              
overcloud_dns_servers: '10.16.36.29'

node_count: 4

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
  - name: compute_0
    flavor: compute
    virtualbmc_port: 6233

topology: >-
  --control-scale 3
  --compute-scale 1

extra_args: >-
  --libvirt-type qemu
  -e /usr/share/openstack-tripleo-heat-templates/environments/puppet-pacemaker.yaml
  --ntp-server pool.ntp.org

run_tempest: false
EOF

bash ./tripleo-quickstart/quickstart.sh \
                --clean \
                --release master \
                --teardown all \
                --tags all \
                -e @$CONFIG \
                $VIRTHOST
```

In the hypervisor run the following command to log-in in
the Undercloud:

```bash
ssh -F /home/toor/.quickstart/ssh.config.ansible undercloud

# Add the TRIPLEO_ROOT var to stackrc 
# to use with tripleo-ci
echo "export TRIPLEO_ROOT=~" >> stackrc

source stackrc
```
At this point you should have your development environment deployed correctly.

Clone the tripleo-ci repo:

```bash
git clone https://github.com/openstack-infra/tripleo-ci
```

And, run the Overcloud pingtest:

```bash
~/tripleo-ci/scripts/tripleo.sh --overcloud-pingtest
```

Enjoy TripleOing (~˘▾˘)~

Note: I had to execute the deployment command 3/4 times to have
an OK deployment, sometimes it just fails (i.e. timeout getting the images).

Note: From the host, `virsh list --all` will work only as the stack user.

Note: Each time you run the quickstart.sh command from the hypervisor
the UC and OC will be nuked (`--teardown all`), you will see tasks like 'PLAY [Tear down undercloud and overcloud vms] **'.

Note: If you delete the Overcloud i.e. using `heat stack-delete overcloud` you can re-deploy what you
had by running the dynamically generated overcloud-deploy.sh script in the stack home folder from the UC.

Note: There are several options for TripleO Quickstart besides the basic 
virthost deployment, check them here: `https://docs.openstack.org/developer/tripleo-quickstart/working-with-extras.html`

<div style="font-size:10px">
  <blockquote>
    <p><strong>Updated 2017/03/17:</strong> Bleh, had to execute several times the deployment command to have it working.. :/ I miss you instack-virt-setup</p>
    <p><strong>Updated 2017/03/16:</strong> The --config option seems to be broken, using instead -e @~/deploy-config.yaml.</p>
    <p><strong>Updated 2017/03/14:</strong> New workflow added.</p>
    <p><strong>Updated 2017/02/27:</strong> Working fine.</p>
    <p><strong>Updated 2017/02/23:</strong> Seems to work.</p>
    <p><strong>Updated 2017/02/23:</strong> instack-virt-setup is deprecatred :( moving to tripleo-quickstart.</p>
  </blockquote>
</div>
