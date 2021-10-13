---
layout: post
title: "Deploying multiple KubeInit clusters in the same hypervisor"
author: "Carlos Camacho"
categories:
  - blog
tags:
- kubernetes
- kubeinit
- cloud
- okd
favorite: false
commentIssueId: 76
refimage: '/static/kubeinit/multiple-clouds-kubeinit.png'
---

The Ansible inventory file defines the hosts and groups of hosts upon which commands,
modules, and tasks in a playbook operate.
In this post it will be explained the way to update the inventory files to allow
deploying multiple KubeInit clusters in the same hypervisor.

>  __*Note 2021/10/13:*__ DEPRECATED - This tutorial only works with
[kubeinit 1.0.2](https://github.com/Kubeinit/kubeinit/releases/tag/1.0.2) make
sure you use this version of the code if you are following this tutorial, or
[refer to the documentation](https://docs.kubeinit.org/) to use the latest code.

# TL;DR;

We will show the required changes to the inventory file to
deploy more than one KubeInit cluster in the same host.

### Basic information

All the change required to achieve the goal of this post will be done in
the same file.

As an example, in this post we will use the [OKD inventory file](https://github.com/Kubeinit/kubeinit/blob/master/kubeinit/hosts/okd/inventory).

### Steps

The following steps are required to deploy a second (or as many are required) KubeInit clusters in the same host.

#### 1. Duplicate the main inventory file.

```
echo "NEW_ID MUST BE AN INTEGER"
new_id=2
cp inventory inventory$new_id
```

#### 2. Adjust network parameters.

The default internal network used is 10.0.0.0/24
so we need to change it to a new range.

We will change from the range 10.0.0 to 10.0.2 (referring to step 1 *new_id=2*)

```
sed -i "s/10\.0\.0/10\.0\.$new_id/g" inventory$new_id
```

#### 3. Adjust the network and bridges names.

We will create new bridges and networks for the new
deployment.

We will change from i.e. kimgtnet0 to kimgtnet2 (referring to step 1 *new_id=2*)

```
sed -i "s/kimgtnet0/kimgtnet$new_id/g" inventory$new_id
sed -i "s/kimgtbr0/kimgtbr$new_id/g" inventory$new_id
sed -i "s/kiextbr0/kiextbr$new_id/g" inventory$new_id
```

#### 4. Replace the hosts MAC addresses for new addresses.

We will randomly replace the MAC addresses for all
guest definitions. The following command will shuffle
the MAC addresses in the file each time is executed.
*Note:* awk does not support hexadecimal number operations,
and it is no possible to replace characters by colons.

```
awk -v seed="$RANDOM" '
  BEGIN { srand(seed) }
  { while(sub(/52:54:00:([[:xdigit:]]{1,2}:){2}[[:xdigit:]]{1,2}/,
              "52,,,54,,,00,,,"int(10+rand()*(99-10+1))",,,"int(10+rand()*(99-10+1))",,,"int(10+rand()*(99-10+1))));
    print > "tmp"
  }
  END { print "MAC shuffled" }
' "inventory$new_id"
mv tmp inventory$new_id
sed -i "s/,,,/:/g" inventory$new_id
```

#### 5. Change the guest names.

VMs are cleaned every time the host is provisioned, if their names are
not updated they will be removed every time.

We will change from okd- to okd2- (referring to step 1 *new_id=2*)

```
sed -i "s/okd-/okd$new_id-/g" inventory$new_id
sed -i "s/clustername0/clustername$new_id/g" inventory$new_id
```

#### 6. Run the deployment command using the new inventory file.

The deployment command should remain exactly as it was,
just update the reference to the new inventory file.

```
# Use the following inventory in your deployment command
-i ./hosts/okd/inventory$new_id
```

#### 7. Cleaning up the host.

Just in case you need to clean things up.

**Warning:** This will destroy any guest in the
host, run with caution.

```
for i in $(virsh -q list | awk '{ print $2 }'); do
  virsh destroy $i;
  virsh undefine $i --remove-all-storage;
done;
```

### Conclusions

Being able to deploy multiple clusters in the same hypervisor it will allow you
to test multiple architectures and scenarios without the need to re-provision
completely the environment.

### The end

If you like this post, please try the code, raise issues, and ask for more details, features or
anything that you feel interested in. Also it would be awesome if you become a stargazer to catch up
updates and new features.

This is the main project [repository](https://github.com/kubeinit/kubeinit).

Happy KubeIniting!
