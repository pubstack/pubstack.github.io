---
layout: post
title: "Running Relax-and-Recover to save your OpenStack deployment"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - cloud
  - tripleo
  - openstack
favorite: true
commentIssueId: 59
refimage: '/static/ReAR_and_OpenStack.png'
---

ReaR is a pretty impressive disaster recovery
solution for Linux. Relax-and-Recover, creates both a
bootable rescue image and a backup of the associated files you choose.

![](/static/ReAR_and_OpenStack.png)

When doing disaster recovery of a system, this Rescue Image plays
the files back from the backup and so in the twinkling of
an eye the latest state.

Various configuration options are available for the rescue image.
For example, slim ISO files, USB sticks or even images for PXE
servers are generated. As many backup options are possible.
Starting with a simple archive file (eg * .tar.gz),
various backup technologies such as IBM Tivoli Storage Manager (TSM),
EMC NetWorker (Legato), Bacula or even Bareos can be addressed.

The ReaR written in Bash enables the skilful
distribution of Rescue Image and if necessary archive file via
NFS, CIFS (SMB) or another transport method in the network.
The actual recovery process then takes place via this transport route.

In this specific case, due to the nature of the OpenStack deployment we will
choose those protocols that are allowed by default in the Iptables rules (SSH, SFTP in particular).

But enough with the theory, here's a practical example of one of many possible configurations.
We will apply this specific use of ReaR to recover
a failed control plane after a critical maintenance task (like an upgrade).

__01 - Prepare the Undercloud backup bucket.__

We need to prepare the place to store the backups from the Overcloud.
From the Undercloud, check you have enough space to make the backups
and prepare the environment. We will also create a user in the
Undercloud with no shell access to be able to push the backups from the
controllers or the compute nodes.

```bash
groupadd backup
mkdir /data
useradd -m -g backup -d /data/backup backup
echo "backup:backup" | chpasswd
chown -R backup:backup /data
chmod -R 755 /data
```

__02 - Run the backup from the Overcloud nodes.__

Let's install some required packages and run some previous
configuration steps.

```bash
#Install packages
sudo yum install rear genisoimage syslinux lftp wget -y

#Make sure you are able to use sshfs to store the ReaR backup
sudo yum install fuse -y
sudo yum groupinstall "Development tools" -y
wget http://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/f/fuse-sshfs-2.10-1.el7.x86_64.rpm
sudo rpm -i fuse-sshfs-2.10-1.el7.x86_64.rpm

sudo mkdir -p /data/backup
sudo sshfs -o allow_other backup@undercloud-0:/data/backup /data/backup
#Use backup password, which is... backup
```

Now, let's configure ReaR config file.

```bash
#Configure ReaR
sudo tee -a "/etc/rear/local.conf" > /dev/null <<'EOF'
OUTPUT=ISO
OUTPUT_URL=sftp://backup:backup@undercloud-0/data/backup/
BACKUP=NETFS
BACKUP_URL=sshfs://backup@undercloud-0/data/backup/
BACKUP_PROG_COMPRESS_OPTIONS=( --gzip )
BACKUP_PROG_COMPRESS_SUFFIX=".gz"
BACKUP_PROG_EXCLUDE=( '/tmp/*' '/data/*' )
EOF
```

Now run the backup, this should create an ISO image in
the Undercloud node (/data/backup/).

**You will be asked for the backup user password**

```bash
sudo rear -d -v mkbackup
```

Now, simulate a failure xD

```
# sudo rm -rf /lib
```

After the ISO image is created, we can proceed to
verify we can restore it from the Hypervisor.

__03 - Prepare the hypervisor.__


```bash
# Enable the use of fusefs for the VMs on the hypervisor
setsebool -P virt_use_fusefs 1

# Install some required packages
sudo yum install -y fuse-sshfs

# Mount the Undercloud backup folder to access the images
mkdir -p /data/backup
sudo sshfs -o allow_other root@undercloud-0:/data/backup /data/backup
ls /data/backup/*
```

__04 - Stop the damaged controller node.__


```bash
virsh shutdown controller-0
# virsh destroy controller-0

# Wait until is down
watch virsh list --all

# Backup the guest definition
virsh dumpxml controller-0 > controller-0.xml
cp controller-0.xml controller-0.xml.bak
```

Now, we need to change the guest definition to boot from the ISO file.

Edit controller-0.xml and update it to boot from the ISO file.

Find the OS section,add the cdrom device and enable the boot menu.

```
<os>
<boot dev='cdrom'/>
<boot dev='hd'/>
<bootmenu enable='yes'/>
</os>
```

Edit the devices section and add the CDROM.

```
<disk type='file' device='cdrom'>
<driver name='qemu' type='raw'/>
<source file='/data/backup/rear-controller-0.iso'/>
<target dev='hdc' bus='ide'/>
<readonly/>
<address type='drive' controller='0' bus='1' target='0' unit='0'/>
</disk>
```

Update the guest definition.

```bash
virsh define controller-0.xml
```

Restart and connect to the guest

```bash
virsh start controller-0
virsh console controller-0
```

You should be able to see the boot menu to start the recover process, select Recover controller-0 and follow the instructions.

![](/static/ReAR1.PNG)

Now, before proceeding to run the controller restore, it's possible that
the host undercloud-0 can't be resolved, just:

```bash
echo "192.168.24.1 undercloud-0" >> /etc/hosts
```

Having resolved the Undercloud host, just follow the wizard, Relax And Recover :)

You yould see a message like:

```
Welcome to Relax-and-Recover. Run "rear recover" to restore your system !

RESCUE controller-0:~ # rear recover
```

![](/static/ReAR2.PNG)

The image restore should progress quickly.

![](/static/ReAR3.PNG)

Continue to see the restore evolution.

![](/static/ReAR4.PNG)

Now, each time you reboot the node will have the ISO file
as the first boot option so it's something we need to fix.
In the mean time let's check if the restore went fine.

Reboot the guest booting from the hard disk.
![](/static/ReAR5.PNG)

Now we can see that the guest VM started successfully.
![](/static/ReAR6.PNG)


Now we need to restore the guest to it's original definition,
so from the Hypervisor we need to restore the `controller-0.xml.bak` 
file we created.

```bash
#From the Hypervisor
virsh shutdown controller-0
watch virsh list --all
virsh define controller-0.xml.bak
virsh start controller-0
```


Enjoy.


## Considerations:

* Space.
* Multiple protocols supported but we might then to update firewall rules, that's why I prefered SFTP.
* Network load when moving data.
* Shutdown/Starting sequence for HA control plane.
* Do we need to backup the data plane?
* User workloads should be handled by a third party backup software.


## Update log:

<div style="font-size:10px">
  <blockquote>
    <p><strong>2019/05/20:</strong> Initial version.</p>
    <p><strong>2019/06/18:</strong> Appeared in <a href="https://superuser.openstack.org/articles/tutorial-rear-openstack-deployment/">OpenStack Superuser blog.</a></p>
  </blockquote>
</div>
