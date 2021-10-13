---
layout: post
title: "Persistent volumes and claims in KubeInit"
author: "Carlos Camacho"
categories:
  - blog
tags:
- kubernetes
- kubeinit
- cloud
- okd
favorite: false
commentIssueId: 73
refimage: '/static/kubeinit/pv/stone_pv.png'
---

Pods are ephemeral and any information stored in them
is not persistent, this means that every time you restart or
create pods from the same application any internal data will be
lost.

>  __*Note 2021/10/13:*__ DEPRECATED - This tutorial only works with
[kubeinit 1.0.2](https://github.com/Kubeinit/kubeinit/releases/tag/1.0.2) make
sure you use this version of the code if you are following this tutorial, or
[refer to the documentation](https://docs.kubeinit.org/) to use the latest code.

The solution for this is to use persistent volumes so pods can persist
their data every time they are restarted, a volume is YAKR (Yet Another Kubernetes Resource).

![](/static/kubeinit/pv/stone_pv.png)

# TL;DR;

We will create a static PV/PVC to be used with an example application.

### Basic information

From [1], lets define some basic concepts.

![](/static/kubeinit/pv/cs_storage_pvc_pv.png)

* Cluster: By default, every cluster is set up with a plug-in to provision file storage.
You can choose to install other add-ons, such as the one for block storage.
To use storage in a cluster, you must create a persistent volume claim, a persistent volume and a physical storage instance.
When you delete the cluster, you have the option to delete related storage instances.

* App: To read from and write to your storage instance, you must mount the persistent volume claim (PVC) to your app.
Different storage types have different read-write rules. For example, you can mount multiple pods to the same PVC for file storage.
Block storage comes with a RWO (ReadWriteOnce) access mode so that you can mount the storage to one pod only.

* Persistent volume claim (PVC): A PVC is the request to provision persistent storage with a specific type and configuration.
To specify the persistent storage flavor that you want, you use Kubernetes storage classes.
The cluster admin can define storage classes.
When you create a PVC, the request is sent to the storage provider.
If the requested configuration does not exist, the storage is not created.

* Persistent volume (PV): A PV is a virtual storage instance that is added as a volume to the cluster.
The PV points to a physical storage device in your account and abstracts the API that is used to communicate with the storage device.
To mount a PV to an app, you must have a matching PVC. Mounted PVs appear as a folder inside the container's file system.

* Physical storage: A physical storage instance that you can use to persist your data.
Examples of physical storage in Kubernetes clusters include File Storage, Block Storage, Object Storage, and local worker node storage.
However, data that is stored on a physical storage instance is not backed up automatically.
Depending on the type of storage that you use, different methods exist to set up backup and restore solutions.

### Static provisioning

If you have an existing persistent storage device,
you can use static provisioning to make the storage instance available to your cluster.

#### How does it work?

Static provisioning is a feature that is native to Kubernetes and that allows cluster administrators
to make existing storage devices available to a cluster. As a cluster administrator, you must know
the details of the storage device, its supported configurations, and mount options.
To make existing storage available to a cluster user, you must manually create the storage device, a PV, and a PVC.

#### Setting up a static PV/PVC in a KubeInit's NFS share

Execute all the following steps from the service machine.

```
# We create a new folder in the main NFS share
mkdir -p /var/nfsshare/test-nfs
chmod -R 777 /var/nfsshare/test-nfs
chown -R nobody:nobody /var/nfsshare/test-nfs

# We define the resources for the PV, PVC, and an example pod to use them
cat << EOF > ~/test_nfs_pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-nfs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: nfs01testnfs
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /var/nfsshare/test-nfs
    server: 10.0.0.100
EOF

cat << EOF > ~/test_nfs_pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-nfs-pvc
spec:
  volumeName: test-nfs-pv
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs01testnfs
  volumeMode: Filesystem
EOF

cat <<EOF > ~/pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-test
  labels:
    name: frontendhttp
spec:
  containers:
    - name: nginx
      image: nginx:1.7.9
      imagePullPolicy: IfNotPresent
      ports:
        - containerPort: 80
          name: http-server
      volumeMounts:
        - mountPath: /var/nfsshare/testmount
          name: pvol
  volumes:
    - name: pvol
      persistentVolumeClaim:
        claimName: test-nfs-pvc
EOF

# We create those resources
export KUBECONFIG=~/install_dir/auth/kubeconfig
oc create -f ~/test_nfs_pv.yaml
oc create -f ~/test_nfs_pvc.yaml
oc create -f ~/pod.yaml

# We test the PV is bound to the PVC
oc get pv
oc get pvc
kubectl get pods
showmount -e 10.0.0.100

# Now we check that our test application is mounting the volume correctly
# As you can see we have the NFS PV mounted in /var/nfsshare/testmount
# we connect to the container and put something in the PV
kubectl exec --stdin --tty nfs-test -- /bin/bash
echo "hello world" > /var/nfsshare/testmount/asdf
exit
cat /var/nfsshare/test-nfs/asdf
# hello world
```

This proves how easy is to create persistent volumes and claims to be used in KubeInit.

### Next steps

Dynamic provisioning is a feature that is native to Kubernetes and that allows a cluster developer
to order storage with a pre-defined type and configuration without knowing all the details about how
to provision the physical storage device. To abstract the details for the specific storage type, the
cluster admin must create storage classes that the developer can use.

Ideally assigning the persistent volumes should be dynamically assigned. In the future
this should be natively be part of KubeInit.

### References

1. [https://cloud.ibm.com/docs/containers?topic=containers-kube_concepts](https://cloud.ibm.com/docs/containers?topic=containers-kube_concepts)

### The end

If you like this post, please try the code, raise issues, and ask for more details, features or
anything that you feel interested in. Also it would be awesome if you become a stargazer to catch up
updates and new features.

This is the main project [repository](https://github.com/kubeinit/kubeinit).

Happy KubeIniting!

---

![](/static/kubeinit/pv/meme.jpg)
