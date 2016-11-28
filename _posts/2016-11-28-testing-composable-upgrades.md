---
layout: post
title: "Testing composable upgrades"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
commentIssueId: 25
---

This is a brief recipe about how I'm testing
composable upgrades N->O.

Based on the original shardy's notes
from [this](http://paste.openstack.org/show/590436/) link.

The following steps are followed to upgrade your Overcloud from Mitaka to latest master (Ocata).

- Deploy latest master TripleO following [this](http://www.anstack.com/blog/2016/07/04/manually-installing-tripleo-recipe.html) post.

- Remove the current Overcloud deployment.

```
  source stackrc
  heat stack-delete overcloud
```

- Remove the Overcloud images and create new ones (for the Newton Overcloud).

```
  cd
  openstack image list
  openstack image delete <image_ID> #Delete all the Overcloud images overcloud-full* 
  rm -rf /home/stack/overcloud-full.*

  export STABLE_RELEASE=newton
  export USE_DELOREAN_TRUNK=1
  export DELOREAN_TRUNK_REPO="http://trunk.rdoproject.org/centos7-newton/current/"
  export DELOREAN_REPO_FILE="delorean.repo"
  /home/stack/tripleo-ci/scripts/tripleo.sh --overcloud-images

  # Or reuse images
  # wget http://buildlogs.centos.org/centos/7/cloud/x86_64/tripleo_images/newton/delorean/overcloud-full.tar
  # tar -xvf overcloud-full.tar
  # openstack overcloud image upload --update-existing
```

- Download Newton tripleo-heat-templates.

```
  cd
  git clone -b stable/newton https://github.com/openstack/tripleo-heat-templates tht-newton
```

- Configure the DNS (needed when upgrading the Overcloud).

```
  neutron subnet-update `neutron subnet-list -f value | awk '{print $1}'` --dns-nameserver 192.168.122.1
```

- Deploy a Newton Overcloud.

```
  openstack overcloud deploy \
  --libvirt-type qemu \
  --templates /home/stack/tht-newton/ \
  -e /home/stack/tht-newton/overcloud-resource-registry-puppet.yaml \
  -e /home/stack/tht-newton/environments/puppet-pacemaker.yaml
```

- Install prerequisites in nodes (if no DNS configured this will fail).

```
cat > upgrade_repos.yaml << EOF
parameter_defaults:
  UpgradeInitCommand: |
    set -e
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
    yum -y install python-heat-agent-ansible
EOF
```

- Download master tripleo-heat-templates, composable upgrades patch and rebase (part of this step will be removed when the composable upgrades with Ansible patch lands).

```
  cd
  git clone https://github.com/openstack/tripleo-heat-templates tht-master
  cd tht-master #Remove after landing this feature
  git review -d 403441 #Remove after landing this feature
  git rebase -i master #Remove after landing this feature
```

- Upgrade Overcloud to master

```
  cd
  openstack overcloud deploy \
  --templates /home/stack/tht-master/ \
  -e /home/stack/tht-master/overcloud-resource-registry-puppet.yaml \
  -e /home/stack/tht-master/environments/puppet-pacemaker.yaml \
  -e /home/stack/tht-master/environments/major-upgrade-composable-steps.yaml \
  -e upgrade_repos.yaml
```

If the last steps manage to finish successfully (WIP), you just have upgraded your Overcloud from Newton to Ocata (latest master).


For more resources related to TripleO deployments, check out the [TripleO YouTube channel](https://www.youtube.com/channel/UCNGDxZGwUELpgaBoLvABsTA).




<div style="font-size:10px">
  <blockquote>
    <p><strong>Updated 2016/11/28:</strong> Not working properly due to an error in the galera cluster (probably related to my env).</p>
  </blockquote>
</div>












```



TASK [setup] *******************************************************************
ok: [localhost]
TASK [Stop cinder_api service (running under httpd)] ***************************
changed: [localhost]
TASK [Stop cinder_scheduler service] *******************************************
changed: [localhost]\n\nTASK [Stop cinder_volume service] **********************************************
changed: [localhost]
TASK [Stop keystone service (running under httpd)] *****************************\nok: [localhost]
TASK [Stop glance_api service] *************************************************\nchanged: [localhost]
TASK [Stop glance_registry service] ********************************************\nchanged: [localhost]
TASK [Stop heat_api service] ***************************************************\nchanged: [localhost]
TASK [Stop heat_api_cfn service] ***********************************************\nchanged: [localhost]
TASK [Stop heat_api_cloudwatch service] ****************************************\nchanged: [localhost]
TASK [Stop heat_engine service] ************************************************\nchanged: [localhost]
TASK [Stop service] ************************************************************\nok: [localhost]
TASK [Stop neutron_dhcp service] ***********************************************\nchanged: [localhost]
TASK [Stop neutron_l3 service] *************************************************\nchanged: [localhost]
TASK [Stop neutron_metadata service] *******************************************\nchanged: [localhost]
TASK [Stop neutron_api service] ************************************************\nchanged: [localhost]
TASK [Stop neutron_ovs_agent service] ******************************************\nchanged: [localhost]
TASK [Stop rabbitmq service] ***************************************************\nok: [localhost]
TASK [Stop nova_conductor service] *********************************************\nchanged: [localhost]
TASK [Stop nova_api service (running under httpd)] *****************************\nok: [localhost]
TASK [Stop nova_scheduler service] *********************************************\nchanged: [localhost]
TASK [Stop nova_consoleauth service] *******************************************\nchanged: [localhost]
TASK [Stop nova_vnc_proxy service] *********************************************\nok: [localhost]\
TASK [Stop openstack-swift-proxy service] **************************************\nchanged: [localhost]


TASK [Stop swift object services] **********************************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: ValueError: need more than 1 value to unpack
fatal: [localhost]: FAILED! => {\"changed\": false, \"failed\": true, \"module_stderr\": \"Traceback (most recent call last):\
  File \\\"/tmp/ansible_9YZa1c/ansible_module_service.py\\\", line 1515, in <module>\
    main()\\
  File \\\"/tmp/ansible_9YZa1c/ansible_module_service.py\\\", line 1471, in main\\
    service.get_service_status()\\
  File \\\"/tmp/ansible_9YZa1c/ansible_module_service.py\\\", line 561, in get_service_status\
    return self.get_systemd_service_status()\\
  File \\\"/tmp/ansible_9YZa1c/ansible_module_service.py\\\", line 542, in get_systemd_service_status\
    d = self.get_systemd_status_dict()\
  File \\\"/tmp/ansible_9YZa1c/ansible_module_service.py\\\", line 518, in get_systemd_status_dict\
    key, value = line.split('=', 1)\
ValueError: need more than 1 value to unpack\
\", \"module_stdout\": \"\", \"msg\": \"MODULE FAILURE\", \"parsed\": false}
NO MORE HOSTS LEFT *************************************************************
\tto retry, use: --limit @/var/lib/heat-config/heat-config-ansible/7585c039-7f38-412a-8392-5ecd9e12fd39_playbook.retry
PLAY RECAP *********************************************************************
localhost                  : ok=24   changed=18   unreachable=0    failed=1   
", 
    "deploy_stderr": "", 

TASK [Stop swift object services] **********************************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: ValueError: need more than 1 value to unpack
fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "module_stderr": "Traceback (most recent call last):\n  File \"/tmp/ansible_ueI7a6/ansible_module_service.py\", line 1515, in <module>\n    main()\n  File \"/tmp/ansible_ueI7a6/ansible_module_service.py\", line 1471, in main\n    service.get_service_status()\n  File \"/tmp/ansible_ueI7a6/ansible_module_service.py\", line 561, in get_service_status\n    return self.get_systemd_service_status()\n  File \"/tmp/ansible_ueI7a6/ansible_module_service.py\", line 542, in get_systemd_service_status\n    d = self.get_systemd_status_dict()\n  File \"/tmp/ansible_ueI7a6/ansible_module_service.py\", line 518, in get_systemd_status_dict\n    key, value = line.split('=', 1)\nValueError: need more than 1 value to unpack\n", "module_stdout": "", "msg": "MODULE FAILURE", "parsed": false}

Fixed 
openstack-swift-object-*
openstack-swift-container-*
openstack-swift-account-*

New error:

2016-11-26 23:47:05Z [AllNodesUpgradeSteps.ControllerUpgrade_Step4.0]: SIGNAL_IN_PROGRESS  Signal: deployment 4b3dfa0c-a1d3-444c-a98b-cc61f5db49f0 failed (2)
2016-11-26 23:47:05Z [AllNodesUpgradeSteps.ControllerUpgrade_Step4.0]: CREATE_FAILED  Error: resources[0]: Deployment to server failed: deploy_status_code : Deployment exited with non-zero status code: 2
2016-11-26 23:47:05Z [AllNodesUpgradeSteps.ControllerUpgrade_Step4]: CREATE_FAILED  Resource CREATE failed: Error: resources[0]: Deployment to server failed: deploy_status_code : Deployment exited with non-zero status code: 2
2016-11-26 23:47:06Z [AllNodesUpgradeSteps.ControllerUpgrade_Step4]: CREATE_FAILED  Error: resources.ControllerUpgrade_Step4.resources[0]: Deployment to server failed: deploy_status_code: Deployment exited with non-zero status code: 2
2016-11-26 23:47:06Z [AllNodesUpgradeSteps]: CREATE_FAILED  Resource CREATE failed: Error: resources.ControllerUpgrade_Step4.resources[0]: Deployment to server failed: deploy_status_code: Deployment exited with non-zero status code: 2
2016-11-26 23:47:07Z [AllNodesUpgradeSteps]: CREATE_FAILED  Error: resources.AllNodesUpgradeSteps.resources.ControllerUpgrade_Step4.resources[0]: Deployment to server failed: deploy_status_code: Deployment exited with non-zero status code: 2
2016-11-26 23:47:07Z [overcloud]: UPDATE_FAILED  Error: resources.AllNodesUpgradeSteps.resources.ControllerUpgrade_Step4.resources[0]: Deployment to server failed: deploy_status_code: Deployment exited with non-zero status code: 2
2016-11-26 23:47:10Z [overcloud-ControllerAllNodesDeployment-zdhtbbghswuq.0]: SIGNAL_COMPLETE  Unknown
2016-11-26 23:47:16Z [overcloud-Controller-hmkrkckqdnez-0-zxwqd4xqp3br.ControllerDeployment]: SIGNAL_COMPLETE  Unknown
2016-11-26 23:47:18Z [overcloud-ControllerHostsDeployment-3mscoxbvr3lu.0]: SIGNAL_COMPLETE  Unknown

"deploy_stdout": "
PLAY [localhost] ***************************************************************
TASK [setup] *******************************************************************
ok: [localhost]
TASK [Start service] ***********************************************************
fatal: [localhost]: FAILED! => {\"changed\": false, \"failed\": true, \"msg\": \"Job for mariadb.service failed because the control process exited with error code. See \\\"systemctl status mariadb.service\\\" and \\\"journalctl -xe\\\" for details.\
\"}
NO MORE HOSTS LEFT *************************************************************
\tto retry, use: --limit @/var/lib/heat-config/heat-config-ansible/45fee150-1bf7-4580-bb3f-a3c7ce60c199_playbook.retry
PLAY RECAP *********************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=1   
", 

become: True for all services restarts
```


