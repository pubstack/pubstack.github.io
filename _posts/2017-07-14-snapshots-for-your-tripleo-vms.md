---
layout: post
title: "Create a TripleO snapshot before breaking it..."
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
commentIssueId: 38
---

The idea of this post is to show how developers can save some time
creating snapshots of their development environments for not
deploying it each time it breaks.

So, don't waste time re-deploying your environment when testing submissions.

I'll show here how to be a little more agile when
deploying your Undercloud/Overcloud for testing purposes.

Deploying a fully working development environment takes
around 3 hours with human supervision...
And breaking it just after deployed is not cool at all...



# Step 1 #

Deploy your environment as usual.



# Step 2 #

Create your Undercloud/Overcloud snapshots.
**Do this as the stack user, otherwise 
virsh won't see the VMs**

```
# The VMs deployed are:
vms=( "undercloud" "control_0" "compute_0" )

# List all VMs
virsh list --all

# List current snapshots
for i in "${vms[@]}"; \
do \
virsh snapshot-list --domain "$i"; \
done

# Dump VMs XLM and check for qemu
for i in "${vms[@]}"; \
do \
virsh dumpxml "$i" | grep -i qemu; \
done

# Create an initial snapshot for each VM
for i in "${vms[@]}"; \
do \
echo "virsh snapshot-create-as --domain $i --name $i-fresh-install --description $i-fresh-install --atomic"; \
virsh snapshot-create-as --domain "$i" --name "$i"-fresh-install --description "$i"-fresh-install --atomic; \
done

# List current snapshots (After they should be already created)
for i in "${vms[@]}"; \
do \
virsh snapshot-list --domain "$i"; \
done

#########################################################################################################
# Current libvirt version does not support live snapshots.
# error: Operation not supported: live disk snapshot not supported with this QEMU binary
# --disk-only and --live not yet available.

# Create the folder for the images
# cd
# mkdir ~/backup_images

# for i in "${vms[@]}"; \
# do \
# echo "<domainsnapshot>" > $i.xml; \
# echo "  <memory snapshot='external' file='/home/stack/backup_images/$i.mem.snap2'/>" >> $i.xml; \
# echo "  <disks>" >> $i.xml; \
# echo "    <disk name='vda'>" >> $i.xml; \
# echo "      <source file='/home/stack/backup_images/$i.disk.snap2'/>" >> $i.xml; \
# echo "    </disk>" >> $i.xml; \
# echo "  </disks>" >> $i.xml; \
# echo "</domainsnapshot>" >> $i.xml; \
# done

# for i in "${vms[@]}"; \
# do \
# echo "virsh snapshot-create $i --xmlfile ~/$i.xml --atomic"; \
# virsh snapshot-create $i --xmlfile ~/$i.xml --atomic; \
# done
```



# Step 3 #

Break your environment xD



# Step 4 #

Restore your snapshots

```
# Commented for safety reasons...
# i=compute_0
i=blehblehbleh
virsh list --all
virsh shutdown $i
sleep 120
virsh list --all
virsh snapshot-revert --domain $i --snapshotname $i-fresh-install --running
virsh list --all
```
