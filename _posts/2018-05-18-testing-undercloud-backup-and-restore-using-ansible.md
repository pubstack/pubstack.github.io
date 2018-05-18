---
layout: post
title: "Testing Undercloud backup and restore using Ansible"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
commentIssueId: 53
---


Testing the Undercloud backup and restore
=========================================

It is possible to test how the Undercloud
backup and restore should be performed using
Ansible.

The following Ansible playbooks will show
how can be used Ansible to test the
backups execution in a test environment.

Creating the Ansible playbooks to run the tasks
-----------------------------------------------

Create a yaml file called uc-backup.yaml
with the following content:

```
---
- hosts: localhost
  tasks:
  - name: Remove any previously created UC backups
    shell: |
      source ~/stackrc
      openstack container delete undercloud-backups --recursive
    ignore_errors: True
  - name: Create UC backup
    shell: |
      source ~/stackrc
      openstack undercloud backup --add-path /etc/ --add-path /root/
```

Create a yaml file called uc-backup-download.yaml
with the following content:

```
---
- hosts: localhost
  tasks:
  - name: Print destroy warning.
    vars:
      msg: |
        We are about to destroy the UC, as we are not
        moving outside the UC the backup tarball, we will
        download it and unzip it in a temporary folder to
        recover the UC using those files.
    debug:
      msg: "{{ msg.split('\n') }}"
  - name: Make sure the temp folder used for the restore does not exist
    become: true
    file:
      path: "/var/tmp/test_bk_down"
      state: absent
  - name: Create temp folder to unzip the backup
    become: true
    file:
      path: "/var/tmp/test_bk_down"
      state: directory
      owner: "stack"
      group: "stack"
      mode: "0775"
      recurse: "yes"
  - name: Download the UC backup to a temporary folder (After breaking the UC we won't be able to get it back)
    shell: |
      source ~/stackrc
      cd /var/tmp/test_bk_down
      openstack container save undercloud-backups
  - name: Unzip the backup
    become: true
    shell: |
      cd /var/tmp/test_bk_down
      tar -xvf UC-backup-*.tar
      gunzip *.gz
      tar -xvf filesystem-*.tar
  - name: Make sure stack user can get the backup files
    become: true
    file:
      path: "/var/tmp/test_bk_down"
      state: directory
      owner: "stack"
      group: "stack"
      mode: "0775"
      recurse: "yes"
```

Create a yaml file called uc-destroy.yaml
with the following content:

```
---
- hosts: localhost
  tasks:
  - name: Remove mariadb
    become: true
    yum: pkg={{ item }}
         state=absent
    with_items:
      - mariadb
      - mariadb-server
  - name: Remove files
    become: true
    file:
      path: "{{ item }}"
      state: absent
    with_items:
      - /root/.my.cnf
      - /var/lib/mysql
```

Create a yaml file called uc-restore.yaml
with the following content:

```
---
- hosts: localhost
  tasks:
    - name: Install mariadb
      become: true
      yum: pkg={{ item }}
           state=present
      with_items:
        - mariadb
        - mariadb-server
    - name: Restart MariaDB
      become: true
      service: name=mariadb state=restarted
    - name: Restore the backup DB
      shell: cat /var/tmp/test_bk_down/all-databases-*.sql | sudo mysql
    - name: Restart MariaDB to perms to refresh
      become: true
      service: name=mariadb state=restarted
    - name: Register root password
      become: true
      shell: cat /var/tmp/test_bk_down/root/.my.cnf | grep -m1 password | cut -d'=' -f2 | tr -d "'"
      register: oldpass
    - name: Clean root password from MariaDB to reinstall the UC
      shell: |
        mysqladmin -u root -p{{ oldpass.stdout }} password ''
    - name: Clean users
      become: true
      mysql_user: name="{{ item }}" host_all="yes" state="absent"
      with_items:
        - ceilometer
        - glance
        - heat
        - ironic
        - keystone
        - neutron
        - nova
        - mistral
        - zaqar
    - name: Reinstall the undercloud
      shell: |
        openstack undercloud install
```


Running the Undercloud backup and restore tasks
-----------------------------------------------

To test the UC backup and restore procedure, run from the UC
after creating the Ansible playbooks:

```
  # This playbook will create the UC backup
  ansible-playbook uc-backup.yaml
  # This playbook will download the UC backup to be used in the restore
  ansible-playbook uc-backup-download.yaml
  # This playbook will destroy the UC (remove DB server, remove DB files, remove config files)
  ansible-playbook uc-destroy.yaml
  # This playbook will reinstall the DB server, restore the DB backup, fix permissions and reinstall the UC
  ansible-playbook uc-restore.yaml
```


Checking the Undercloud state
-----------------------------

After finishing the Undercloud restore playbook the user should be able to execute again
any CLI command like:

```
  source ~/stackrc
  openstack stack list
```

Source code available in [GitHub](https://github.com/ccamacho/tripleo-ansible/tree/master/undercloud-backup-restore-check)
