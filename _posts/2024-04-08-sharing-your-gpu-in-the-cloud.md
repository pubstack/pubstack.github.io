---
layout: post
title: "Sharing your GPU in the cloud"
author: "Carlos Camacho"
categories:
  - blog
tags:
  # - draft
  - cloud
  - kubernetes
  - engineering
  - openshift
# hidden: true
favorite: true
commentIssueId: 93
refimage: '/static/gpu_post/cloud.jpg'
---

In this post we will introduce different methods for sharing GPU resources
across workloads in K8s clusters.

![](/static/gpu_post/cloud_lego.jpg)

GPU sharing refers to the practice of allowing multiple users or processes
to access and utilize the resources of a single Graphics Processing Unit (GPU) concurrently.
This approach can be beneficial in scenarios where there are multiple workloads
that can utilize GPU resources intermittently or simultaneously, such as in cloud
computing environments, data centers, or research institutions.

### Prerequisites:

- OpenShift 4.15 deployed.
- NFD operator.
- Nvidia GPU operator from master.

<div style="float: right; width: 230px; background: white;"><img src="/static/gpu_post/slice.png" alt="" style="border:15px solid #FFF"></div>

Time-slicing, is a technique used in GPU resource management where the available
GPU resources are divided into time intervals, or "slices," and allocated to different
users or processes sequentially. Each user or process is granted access to the GPU
for a specified duration, known as a time slice, before the GPU is relinquished and
made available to the next user or process in the queue.
This method of GPU sharing is particularly useful in environments where there are
multiple users or processes vying for access to limited GPU resources. By allocating
GPU time slices to different users or processes, time-slicing ensures fair and efficient
utilization of the GPU among multiple competing workloads.

<div style="float: left; width: 230px; background: white;"><img src="/static/gpu_post/instance.jpg" alt="" style="border:15px solid #FFF"></div>

Multi-Instance GPU (MIG), revolutionizes GPU utilization in data center environments
by allowing a single physical GPU to be partitioned into multiple isolated instances,
each with its own dedicated compute resources, memory, and performance profiles. MIG
enables efficient sharing of GPU resources among multiple users or workloads by
providing predictable performance guarantees and workload isolation. With MIG,
administrators can dynamically adjust the number and size of MIG instances to adapt
to changing workload demands, ensuring optimal resource utilization and scalability.
This innovative technology enhances flexibility, efficiency, and performance in
GPU-accelerated computing environments, enabling organizations to meet the diverse
needs of applications and users while maximizing GPU utilization.

<div style="float: right; width: 230px; background: white;"><img src="/static/gpu_post/semaphore.png" alt="" style="border:15px solid #FFF"></div>

Multi-Process Service (MPS), facilitates the concurrent sharing of a single
GPU among multiple CUDA applications. By allowing the GPU to swiftly transition
between various CUDA contexts, MPS optimizes GPU resource utilization across
multiple processes. This capability enables efficient allocation of GPU resources
to different applications running simultaneously, ensuring that the GPU is
effectively utilized even when serving multiple workloads concurrently. MPS
enhances GPU efficiency by dynamically managing CUDA contexts, enabling seamless
context switching and minimizing overhead, thus maximizing GPU throughput and
responsiveness in multi-application environments.

<div style="display: flex; justify-content: center;">
  <div style="width: 230px; background: white;"><img src="/static/gpu_post/gpu_lego.png" alt="" style="border:15px solid #FFF"></div>
</div>

Overall, these techniques aim to improve the efficiency of GPU utilization and
accommodate the diverse needs of users or processes sharing the GPU resources.

A shortcut to the different strategies reviewed as follows:

### Strategies

1. [Quick check](#quick-check)
2. [Enabling Time-slicing](#time-slicing)
3. [Enabling Multi-Instance GPU (MIG)](#mig)
4. [Enabling MPS](#mps)
5. [Reset](#reset)

## Quick check <a name="quick-check"></a>

Let's check before anything that the GPU is detected and configured before continuing.

```bash
NODE=perf-intel-6.perf.eng.bos2.dc.redhat.com
kubectl label --list nodes $NODE | \
  grep nvidia.com 
```

## Enabling Time-slicing <a name="time-slicing"></a>

<div style="float: right; width: 150px; background: white;"><img src="/static/gpu_post/icon.png" alt="" style="border:15px solid #FFF"></div>

The following CR example defines how we will be sharing the GPU
in a config map (this wont have any effect in the cluster at the moment).

```bash
cat << EOF | kubectl apply -f -
---
apiVersion: v1
kind: ConfigMap
metadata:
  # name: device-plugin-config # NVIDIA
  name: time-slicing-config #OpenShift
  namespace: nvidia-gpu-operator
data:
  NVIDIA-A100-PCIE-40GB: |-
    version: v1
    sharing:
      timeSlicing:
        resources:
          - name: nvidia.com/gpu
            replicas: 10
EOF
```

With the resource created we need to patch the initial ClusterPolicy
from the GPU operator `gpu-cluster-policy`.

```bash
oc patch clusterpolicy \
    gpu-cluster-policy \
    -n nvidia-gpu-operator \
    --type merge \
    -p '{"spec": {"devicePlugin": {"config": {"name": "time-slicing-config"}}}}'
```

The previous example will update the `devicePlugin` configuration in the `gpu-cluster-policy`.
When inspecting the cluster policy it should look like

```bash
devicePlugin:
  config:
    default: ''
    name: time-slicing-config
  enabled: true
  mps:
    root: /run/nvidia/mps
```

So we fetch the configuration from the previously created config map `time-slicing-config`.

To make sure the NFD operator runs we label the node stating that the `device-plugin.config` should point to `NVIDIA-A100-PCIE-40GB`. And we label the node we want to apply the configuration

```bash
oc label \
  --overwrite node perf-intel-6.perf.eng.bos2.dc.redhat.com \
  nvidia.com/device-plugin.config=NVIDIA-A100-PCIE-40GB
```

After a few minutes we can change that the NFD operator reconfigured the node to use time-slicing

```bash
oc get node \
  --selector=nvidia.com/gpu.product=NVIDIA-A100-PCIE-40GB-SHARED \
  -o json | jq '.items[0].status.capacity'
```

```bash
{
  "cpu": "128",
  "ephemeral-storage": "3123565732Ki",
  "hugepages-1Gi": "0",
  "hugepages-2Mi": "0",
  "memory": "527845520Ki",
  "nvidia.com/gpu": "8",
  "pods": "250"
}
```

Now we test we can schedule the shared GPU to a pod (or many slices).
Note that we allocate the pod with the limits instead of the nodeselector
because the GPU label name changed, so we assume we dont know the name.

```bash
cat << EOF | kubectl create -f -
---
apiVersion: v1
kind: Pod
metadata:
  # generateName: command-nvidia-smi-
  name: command-nvidia-smi
spec:
  restartPolicy: Never
  containers:
    - name: cuda-container
      image: nvcr.io/nvidia/cuda:12.1.0-base-ubi8
      command: ["/bin/sh","-c"]
      args: ["nvidia-smi"]
      resources:
        limits:
          nvidia.com/gpu: 1 # requesting 1 GPU
  # nodeSelector:
  #   nvidia.com/gpu.product: NVIDIA-A100-PCIE-40GB
EOF
```

```bash
pod/command-nvidia-smi created
ccamacho@guateque:~/dev/$ kubectl logs command-nvidia-smi
Mon Apr  8 10:01:26 2024       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.54.15              Driver Version: 550.54.15      CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA A100-PCIE-40GB          On  |   00000000:0A:00.0 Off |                    0 |
| N/A   35C    P0             38W /  250W |   39023MiB /  40960MiB |      0%      Default |
|                                         |                        |             Disabled |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
+-----------------------------------------------------------------------------------------+
```

In the previous case a pod will be scheduled and its execution will end, now let's see
how can we schedule the 8 slices that were created before.

We can create a deployment with 11 pods that will run
[dcgmproftester12](https://docs.nvidia.com/datacenter/dcgm/latest/user-guide/feature-overview.html#cuda-test-generator-dcgmproftester).
In this case dcgmproftester12 will generate load as a half-precision
matrix-multiply-accumulate for the Tensor Cores (-t 1004) for 5 minutes (-d 300).

```bash
cat << EOF | kubectl apply -f -
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nvidia-plugin-test
  labels:
    app: nvidia-plugin-test
spec:
  replicas: 11
  selector:
    matchLabels:
      app: nvidia-plugin-test
  template:
    metadata:
      labels:
        app: nvidia-plugin-test
    spec:
      tolerations:
        - key: nvidia.com/gpu
          operator: Exists
          effect: NoSchedule
      containers:
       - name: dcgmproftester12
         image: nvcr.io/nvidia/cloud-native/dcgm:3.3.5-1-ubi9
         # What is the difference between nvcr.io/nvidia/cloud-native and nvcr.io/nvidia/k8s?
         command: ["/bin/sh", "-c"]
         args:
            - while true; do /usr/bin/dcgmproftester12 --no-dcgm-validation -t 1004 -d 30; sleep 30; done
         resources:
           limits:
             # To check gpu vs gpu.shared (MPS)
             nvidia.com/gpu: "1"
         securityContext:
           privileged: true
         # The following SYS_ADMIN capability is not enough  
         # securityContext:
         #   capabilities:
         #     add: ["SYS_ADMIN"]
EOF
```

Where we have 8 from those 8 slices that were created used (8 running and 3 pending).

```bash
ccamacho@guateque:~/dev$ kubectl get pods --selector=app=nvidia-plugin-test
NAME                                  READY   STATUS    RESTARTS   AGE
nvidia-plugin-test-6988cc8f8f-22gt2   1/1     Running   0          53s
nvidia-plugin-test-6988cc8f8f-45dgd   1/1     Running   0          53s
nvidia-plugin-test-6988cc8f8f-gjrg8   1/1     Running   0          53s
nvidia-plugin-test-6988cc8f8f-lwlvh   1/1     Running   0          53s
nvidia-plugin-test-6988cc8f8f-rq6hx   1/1     Running   0          53s
nvidia-plugin-test-6988cc8f8f-tgkxm   1/1     Running   0          53s
nvidia-plugin-test-6988cc8f8f-wdc7l   1/1     Running   0          53s
nvidia-plugin-test-6988cc8f8f-wmzz9   1/1     Running   0          53s
nvidia-plugin-test-6988cc8f8f-wn8jt   0/1     Pending   0          53s
nvidia-plugin-test-6988cc8f8f-x5lbp   0/1     Pending   0          53s
nvidia-plugin-test-6988cc8f8f-zdvwn   0/1     Pending   0          53s
```

If `is forbidden: unable to validate against any security context constraint: [spec.initContainers[0].securityContext.privileged: Invalid value: true: Privileged containers are not allowed spec.init`

```bash
oc adm policy add-scc-to-user privileged -z default -n <namespace>
```

After `dcgmproftester12` finishes, the pods logs show:

```bash
Skipping CreateDcgmGroups() since DCGM validation is disabled
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.78e+05 gflops).
.
.
.
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.89e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.89e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.89e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.89e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.89e+05 gflops).
Worker 0:0[1004]: Message: Bus ID 00000000:0A:00.0 mapped to cuda device ID 0
DCGM CudaContext Init completed successfully.
CU_DEVICE_ATTRIBUTE_MAX_THREADS_PER_MULTIPROCESSOR: 2048
CUDA_VISIBLE_DEVICES: GPU-539a31f4-1487-3a42-cc55-910b7bf82d0c
CU_DEVICE_ATTRIBUTE_MULTIPROCESSOR_COUNT: 108
CU_DEVICE_ATTRIBUTE_MAX_SHARED_MEMORY_PER_MULTIPROCESSOR: 167936
CU_DEVICE_ATTRIBUTE_COMPUTE_CAPABILITY_MAJOR: 8
CU_DEVICE_ATTRIBUTE_COMPUTE_CAPABILITY_MINOR: 0
CU_DEVICE_ATTRIBUTE_GLOBAL_MEMORY_BUS_WIDTH: 5120
CU_DEVICE_ATTRIBUTE_MEMORY_CLOCK_RATE: 1215
Max Memory bandwidth: 1555200000000 bytes (1555.2 GiB)
CU_DEVICE_ATTRIBUTE_ECC_SUPPORT: true
```

```bash
oc rsh \
  -n nvidia-gpu-operator \
  $(kubectl get pods -n nvidia-gpu-operator | grep -E 'nvidia-dcgm-exporter.*' | awk '{print $1}') nvidia-smi
```

## Enabling Multi-Instance GPU (MIG) <a name="mig"></a>

<div style="float: right; width: 150px; background: white;"><img src="/static/gpu_post/motherboard_gpu.png" alt="" style="border:15px solid #FFF"></div>

Similarly as time-slicing the configuration needs to be adjusted using both configmaps
and updating the cluster policy to the changes are applied correctly. The labeling
step done in time-slicing needs to be executed only once.

Depending on the GPU different [MIG partitions can be created](https://docs.nvidia.com/datacenter/tesla/mig-user-guide/index.html):

| Product        | Architecture   | Microarchitecture | Compute Capability | Memory Size | Max Number of Instances |
|----------------|----------------|-------------------|---------------------|-------------|-------------------------|
| H100-SXM5      | Hopper         | GH100             | 9.0                 | 80GB        | 7                       |
| H100-PCIE      | Hopper         | GH100             | 9.0                 | 80GB        | 7                       |
| H100-SXM5      | Hopper         | GH100             | 9.0                 | 94GB        | 7                       |
| H100-PCIE      | Hopper         | GH100             | 9.0                 | 94GB        | 7                       |
| H100 on GH200  | Hopper         | GH100             | 9.0                 | 96GB        | 7                       |
| A100-SXM4      | NVIDIA Ampere  | GA100             | 8.0                 | 40GB        | 7                       |
| A100-SXM4      | NVIDIA Ampere  | GA100             | 8.0                 | 80GB        | 7                       |
| A100-PCIE      | NVIDIA Ampere  | GA100             | 8.0                 | 40GB        | 7                       |
| A100-PCIE      | NVIDIA Ampere  | GA100             | 8.0                 | 80GB        | 7                       |
| A30            | NVIDIA Ampere  | GA100             | 8.0                 | 24GB        | 4                       |

Where in the case of the A100 we can have the following profiles:

| Profile   | Memory | Compute Units | Maximum number of homogeneous instances |
|-----------|--------|---------------|----------------------------------------|
| 1g.5gb    | 5 GB   | 1             | 7                                      |
| 2g.10gb   | 10 GB  | 2             | 3                                      |
| 3g.20gb   | 20 GB  | 3             | 2                                      |
| 4g.20gb   | 20 GB  | 4             | 1                                      |
| 7g.40gb   | 40 GB  | 7             | 1                                      |

We can check that by running:

```bash
oc rsh \
  -n nvidia-gpu-operator \
  $(kubectl get pods -n nvidia-gpu-operator | grep -E 'nvidia-dcgm-exporter.*' | awk '{print $1}') nvidia-smi mig -lgip
```

Where the profiles are described with the notation `<COMPUTE>.<MEMORY>`, an administrator
will create a set of profiles that can be consumed by the workloads.

And let's check how many MIG-enabled are available.

```bash
oc rsh \
  -n nvidia-gpu-operator \
  $(kubectl get pods -n nvidia-gpu-operator | grep -E 'nvidia-driver-daemonset.*' | awk '{print $1}') nvidia-smi mig -lgi
```

This should output `No MIG-enabled devices found.` because we didn't create any device yet.

[The strategies could be](https://docs.nvidia.com/datacenter/cloud-native/openshift/latest/mig-ocp.html#configuring-mig-devices-in-openshift) (it is important that `A GEOMETRY MUST BE DEFINED PER CLUSTER`)
single, all GPUs within the same node with the same geometry i.e. 1g.5gb.
mixed, heterogeneous advertisement like single node with multiple GPUs, each GPU can be configured in a different MIG geometry.

Let see our MIG related labels

```bash
kubectl get node -o json | \
   jq '.items[0].metadata.labels | with_entries(select(.key | startswith("nvidia.com")))'
``` 

Where the MIG related labels are:

```bash
.
.
.
  "nvidia.com/mig.capable": "true",
  "nvidia.com/mig.config": "all-disabled",
  "nvidia.com/mig.config.state": "success",
  "nvidia.com/mig.strategy": "single",
  "nvidia.com/mps.capable": "false"
```

```bash
cat << EOF | kubectl apply -f -
---
apiVersion: v1
kind: ConfigMap
metadata:
  # name: device-plugin-config # NVIDIA
  name: mig-profiles-config #OpenShift
  namespace: nvidia-gpu-operator
data:
  NVIDIA-A100-PCIE-40GB: |-
    version: v1
    sharing:
      timeSlicing:
        resources:
          - name: nvidia.com/gpu
            replicas: 2
          - name: nvidia.com/mig-1g.5gb
            replicas: 2
          - name: nvidia.com/mig-2g.10gb
            replicas: 2
          - name: nvidia.com/mig-3g.20gb
            replicas: 2
          - name: nvidia.com/mig-7g.40gb
            replicas: 2
EOF
```

```bash
oc patch clusterpolicy gpu-cluster-policy \
    -n nvidia-gpu-operator --type merge \
    -p '{"spec": {"devicePlugin": {"config": {"name": "mig-profiles-config"}}}}'
```

The device config plugin already points to this GPU name `NVIDIA-A100-PCIE-40GB`,
when we configured timeslicing.
```bash
oc label \
  --overwrite node perf-intel-6.perf.eng.bos2.dc.redhat.com \
  nvidia.com/device-plugin.config=NVIDIA-A100-PCIE-40GB
```

We patch the cluster policy to enable a `mixed` strategy.

```bash
STRATEGY=mixed
oc patch clusterpolicy/gpu-cluster-policy \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/mig/strategy", "value": '$STRATEGY'}]'
```

We apply the MIG Partitioning profiles, you can relabel as you need to change between
different MIG profiles:

```bash
NODE_NAME=perf-intel-6.perf.eng.bos2.dc.redhat.com
# MIG_CONFIGURATION=all-1g.5gb
#MIG_CONFIGURATION=all-2g.10gb
#MIG_CONFIGURATION=all-4g.20gb
MIG_CONFIGURATION=all-3g.20gb
#MIG_CONFIGURATION=all-7g.40gb
oc label node/$NODE_NAME nvidia.com/mig.config=$MIG_CONFIGURATION --overwrite=true

# or
kubectl label nodes $NODE_NAME nvidia.com/mig.config=$MIG_CONFIGURATION --overwrite 
```

Wait for the mig-manager to perform the reconfiguration:

```bash
oc -n nvidia-gpu-operator logs ds/nvidia-mig-manager --all-containers -f --prefix
```

Get all the MIG-enabled devices.

```bash
oc rsh \
  -n nvidia-gpu-operator \
  $(kubectl get pods -n nvidia-gpu-operator | grep -E 'nvidia-driver-daemonset.*' | awk '{print $1}') nvidia-smi mig -lgi
```

If this return `No MIG-enabled devices found.` there is something wrong that needs to be debugged. 

Otherwise:

```bash
+-------------------------------------------------------+
| GPU instances:                                        |
| GPU   Name             Profile  Instance   Placement  |
|                          ID       ID       Start:Size |
|=======================================================|
|   0  MIG 3g.20gb          9        1          4:4     |
+-------------------------------------------------------+
|   0  MIG 3g.20gb          9        2          0:4     |
+-------------------------------------------------------+
```

Make sure the capabilities match

```bash
kubectl describe nodes perf-intel-6.perf.eng.bos2.dc.redhat.com | grep "nvidia.com/"
```

See that we stop using `nvidia.com/gpu` for `mig-3g.20gb` instead.

```bash
                    nvidia.com/mig-3g.20gb.slices.gi=3
                    nvidia.com/mig.capable=true
                    nvidia.com/mig.config=all-3g.20gb
                    nvidia.com/mig.config.state=success
                    nvidia.com/mig.strategy=mixed
                    nvidia.com/mps.capable=false
                    nvidia.com/gpu-driver-upgrade-enabled: true
  nvidia.com/gpu:          0
  nvidia.com/mig-3g.20gb:  4
  nvidia.com/gpu:          0
  nvidia.com/mig-3g.20gb:  4
  nvidia.com/gpu          0             0
  nvidia.com/mig-3g.20gb  0             0
```

Lets test it:

```bash
cat << EOF | kubectl apply -f -
---
apiVersion: v1
kind: Pod
metadata:
  name: test-1
  namespace: model-serving-test
spec:
  restartPolicy: OnFailure
  containers:
  - name: gpu-test
    image: nvcr.io/nvidia/pytorch:23.07-py3
    command: ["nvidia-smi"]
    args: ["-L"]
    resources:
      limits:
        nvidia.com/mig-3g.20gb: 1
EOF
```

The pod should be scheduled and to show in the logs something like:

```bash
GPU 0: NVIDIA A100-PCIE-40GB (UUID: GPU-539a31f4-1487-3a42-cc55-910b7bf82d0c)
MIG 3g.20gb Device 0: (UUID: MIG-e3864840-4bd8-5b03-9ed2-59a7f1d73b5f)
```

## Enabling MPS <a name="mps"></a>

<div style="float: right; width: 150px; background: white;"><img src="/static/gpu_post/multi.png" alt="" style="border:15px solid #FFF"></div>

We go through a similar process as enabling time-slicing.
We create a config map with the MPS configuration.
The labeling step done in time-slicing needs to be executed only once.

```bash
cat << EOF | kubectl apply -f -
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: mps-enable-config #OpenShift
  namespace: nvidia-gpu-operator
data:
  NVIDIA-A100-PCIE-40GB: |-
    version: v1
    sharing:
      mps:
        # renameByDefault: false
        # if we rename the resources they will move
        # to gpu.shared vs gpu
        resources:
        - name: nvidia.com/gpu
          replicas: 10
EOF
```

And we path the cluster policy to apply the configuration from the new config map. Now we patch the CR

```bash
oc patch clusterpolicy \
    gpu-cluster-policy \
    -n nvidia-gpu-operator \
    --type merge \
    -p '{"spec": {"devicePlugin": {"config": {"name": "mps-enable-config"}}}}'
```

With the cluster policy patched, we dont have to relabel the node because it was done in a previous step:
```bash
oc label \
  --overwrite node perf-intel-6.perf.eng.bos2.dc.redhat.com \
  nvidia.com/device-plugin.config=NVIDIA-A100-PCIE-40GB
```

To trigger the NFD `gpu-feature-discovery-XXX` POD `YOU NEED TO MAKE SURE THE CLUSTER POLICY IS PATCHED`.
Now the product name should change from -SHARED (from time slicing) to the original name.


```bash
oc get node \
  --selector=nvidia.com/gpu.product=NVIDIA-A100-PCIE-40GB \
  -o json | jq '.items[0].status.capacity'
```

And we query the gpu labels.

```bash
kubectl describe node perf-intel-6.perf.eng.bos2.dc.redhat.com | awk '/^Labels:/,/^Taints:/' | grep nvidia.com
```

And these should be available:

```bash
nvidia.com/gpu.sharing-strategy=mps
nvidia.com/mps.capable=true
```
The MPS Control Daemon should be up and running:

```bash
pod_name=$(kubectl get pods -n nvidia-gpu-operator -o=name | grep 'nvidia-device-plugin-mps-control-daemon-' | tail -n 1 | cut -d '/' -f 2)
kubectl logs -n nvidia-gpu-operator $pod_name
```

```bash
nvidia-device-plugin-mps-control-daemon-9ff2n
I0405 12:49:04.996976 46 main.go:78] Starting NVIDIA MPS Control Daemon d838ad11
commit: d838ad11d323a71f19c136fabbf65c9e23b2ae81
I0405 12:49:04.997136 46 main.go:55] "Starting NVIDIA MPS Control Daemon" version=<
d838ad11
commit: d838ad11d323a71f19c136fabbf65c9e23b2ae81
>
I0405 12:49:04.997150 46 main.go:107] Starting OS watcher.
I0405 12:49:04.997615 46 main.go:121] Starting Daemons.
I0405 12:49:04.997654 46 main.go:164] Loading configuration.
I0405 12:49:04.998024 46 main.go:172] Updating config with default resource matching patterns.
I0405 12:49:04.998064 46 main.go:183]
Running with config:
{
"version": "v1",
"flags": {
"migStrategy": "single",
"failOnInitError": null,
"gdsEnabled": null,
"mofedEnabled": null,
"useNodeFeatureAPI": null,
"plugin": {
"passDeviceSpecs": null,
"deviceListStrategy": null,
"deviceIDStrategy": null,
"cdiAnnotationPrefix": null,
"nvidiaCTKPath": null,
"containerDriverRoot": null
}
},
"resources": {
"gpus": [
{
"pattern": "*",
"name": "nvidia.com/gpu"
}
],
"mig": [
{
"pattern": "*",
"name": "nvidia.com/gpu"
}
]
},
"sharing": {
"timeSlicing": {},
"mps": {
"renameByDefault": true,
"failRequestsGreaterThanOne": true,
"resources": [
{
"name": "nvidia.com/gpu",
"rename": "nvidia.com/gpu.shared",
"devices": "all",
"replicas": 10
}
]
}
}
}
I0405 12:49:04.998071 46 main.go:187] Retrieving MPS daemons.
I0405 12:49:05.073068 46 daemon.go:93] "Staring MPS daemon" resource="nvidia.com/gpu.shared"
I0405 12:49:05.113709 46 daemon.go:131] "Starting log tailer" resource="nvidia.com/gpu.shared"
[2024-04-05 12:49:05.086 Control 61] Starting control daemon using socket /mps/nvidia.com/gpu.shared/pipe/control
[2024-04-05 12:49:05.086 Control 61] To connect CUDA applications to this daemon, set env CUDA_MPS_PIPE_DIRECTORY=/mps/nvidia.com/gpu.shared/pipe
[2024-04-05 12:49:05.100 Control 61] Accepting connection...
[2024-04-05 12:49:05.100 Control 61] NEW UI
[2024-04-05 12:49:05.100 Control 61] Cmd:set_default_device_pinned_mem_limit 0 4096M
[2024-04-05 12:49:05.100 Control 61] UI closed
[2024-04-05 12:49:05.113 Control 61] Accepting connection...
[2024-04-05 12:49:05.113 Control 61] NEW UI
[2024-04-05 12:49:05.113 Control 61] Cmd:set_default_active_thread_percentage 10
[2024-04-05 12:49:05.113 Control 61] 10.0
[2024-04-05 12:49:05.113 Control 61] UI closed
```

Make sure the capabilities match

```bash
kubectl describe nodes perf-intel-6.perf.eng.bos2.dc.redhat.com | grep "nvidia.com/"
```

```bash
                    nvidia.com/device-plugin.config=NVIDIA-A100-PCIE-40GB
                    nvidia.com/gfd.timestamp=1715175821
                    nvidia.com/gpu.product=NVIDIA-A100-PCIE-40GB
                    nvidia.com/gpu.replicas=10
                    nvidia.com/gpu.sharing-strategy=mps
  nvidia.com/gpu:         2
  nvidia.com/gpu.shared:  20
  nvidia.com/gpu:         0
  nvidia.com/gpu.shared:  20
  nvidia.com/gpu         0             0
  nvidia.com/gpu.shared  0             0

```

Lets see how to schedule a pod using mps:

```bash
cat << EOF | kubectl create -f -
---
apiVersion: v1
kind: Pod
metadata:
  generateName: command-nvidia-smi-
  # name: command-nvidia-smi
spec:
  hostIPC: true # 
  securityContext:
    runAsUser: 1000 # 
  restartPolicy: Never
  containers:
    - name: cuda-container
      image: nvcr.io/nvidia/cuda:12.1.0-base-ubi8
      command: ["/bin/sh","-c"]
      args: ["nvidia-smi"]
      resources:
        limits:
          nvidia.com/gpu: "1"
  # nodeSelector:
  #   nvidia.com/gpu.product: NVIDIA-A100-PCIE-40GB
EOF
```

```bash
cat << EOF | kubectl create -f -
---
apiVersion: v1
kind: Pod
metadata:
  generateName: dcgmproftester12-
  # name: dcgmproftester12
  namespace: nvidia-gpu-operator
spec:
  hostIPC: true # 
  securityContext:
    runAsUser: 0 # 
  restartPolicy: Never
  containers:
    - name: dcgmproftester12
      image: nvcr.io/nvidia/cloud-native/dcgm:3.3.5-1-ubi9
      command: ["/bin/sh", "-c"]
      args:
        - while true; do /usr/bin/dcgmproftester12 --no-dcgm-validation -t 1004 -d 300; sleep 30; done
      securityContext:
        privileged: true
      # The following SYS_ADMIN capability is not enough  
        capabilities:
          add: ["SYS_ADMIN"]
      resources:
        limits:
          nvidia.com/gpu: "1"
  # nodeSelector:
  #   nvidia.com/gpu.product: NVIDIA-A100-PCIE-40GB
EOF
```

From the host:

```bash
sudo chroot /run/nvidia/driver nvidia-smi
tail -f /run/nvidia/mps/nvidia.com/gpu/log/server.log
```

If it works you should see something like:


```bash
sudo chroot /run/nvidia/driver nvidia-smi
Thu May  9 18:01:08 2024       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.54.15              Driver Version: 550.54.15      CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA A100-PCIE-40GB          On  |   00000000:0A:00.0 Off |                    0 |
| N/A   55C    P0            121W /  250W |     495MiB /  40960MiB |    100%   E. Process |
|                                         |                        |             Disabled |
+-----------------------------------------+------------------------+----------------------+
|   1  NVIDIA A100-PCIE-40GB          On  |   00000000:AE:00.0 Off |                    0 |
| N/A   38C    P0             62W /  250W |      38MiB /  40960MiB |      0%   E. Process |
|                                         |                        |             Disabled |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A    543063      C   nvidia-cuda-mps-server                         30MiB |
|    0   N/A  N/A    686657    M+C   /usr/bin/dcgmproftester12                     456MiB |
|    1   N/A  N/A    543063      C   nvidia-cuda-mps-server                         30MiB |
+-----------------------------------------------------------------------------------------+
```

Where the dcgmproftester12 and nvidia-cuda-mps-server are running. 

And a deployment to schedule multiple pods:

```bash
cat << EOF | kubectl apply -f -
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nvidia-plugin-test
  labels:
    app: nvidia-plugin-test
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nvidia-plugin-test
  template:
    metadata:
      labels:
        app: nvidia-plugin-test
    spec:
      tolerations:
        - key: nvidia.com/gpu.shared
          operator: Exists
          effect: NoSchedule
      containers:
       - name: dcgmproftester12
         image: nvcr.io/nvidia/cloud-native/dcgm:3.3.5-1-ubi9
         # What is the difference between nvcr.io/nvidia/cloud-native and nvcr.io/nvidia/k8s?
         command: ["/bin/sh", "-c"]
         args:
            - while true; do /usr/bin/dcgmproftester12 --no-dcgm-validation -t 1004 -d 30; sleep 30; done
         resources:
           limits:
             nvidia.com/gpu: "1"
         securityContext:
           privileged: true
         # The following SYS_ADMIN capability is not enough  
         # securityContext:
         #   capabilities:
         #     add: ["SYS_ADMIN"]
EOF
```

## Reset <a name="reset"></a>

To reset the cluster to un-share the GPUs proceed to:

Make sure pods are not using the GPU. The following snippet will
fetch all pods from any namespace requesting a GPU,from the pod spec
`resources.limits.nvidia\.com/gpu: "1"`.

```bash
kubectl get pods --all-namespaces -o=jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\n"}{end}' \
| while read -r namespace pod; do \
    kubectl get pod "$pod" -n "$namespace" -o=jsonpath='{range .spec.containers[*]}{.name}{"\t"}{.resources.limits.nvidia\.com/gpu}{"\n"}{end}' \
    | grep '1' >/dev/null && \
    echo "$namespace\t$pod"; \
  done
```

The previous command shouldn't return any pod name,
otherwise you have pods requesting GPU access.

Recreate the NVIDIA GPU operator cluster policy.

```bash
oc get clusterpolicies gpu-cluster-policy -o yaml
oc delete clusterpolicy gpu-cluster-policy

#oc get nodefeaturediscoveries nfd-instance -o yaml -n openshift-nfd
#oc delete nodefeaturediscovery nfd-instance -n openshift-nfd

#NODE=perf-intel-6.perf.eng.bos2.dc.redhat.com
#kubectl label --list nodes $NODE | \
#  grep nvidia.com | \
#  awk -F= '{print $1}' | xargs -I{} kubectl label node $NODE {}-
```

<!--
- Create both the NFD nodefeaturediscovery and the Nvidia clusterpolicy
- Make sure labels are consistent with an initialized GPU
-->

```bash
kubectl label --list nodes $NODE | grep nvidia.com
```

Create a new NVIDIA GPU operator cluster policy.


<!--
- Remove the nodefeaturediscovery.
- Remove the clusterpolicy.
- Remove the node labels.
- Create a new nodefeaturediscovery and a new clusterpolicy.
-->
## Update log:

<div style="font-size:10px">
  <blockquote>
    <p><strong>2024/04/04:</strong> Initial draft.</p>
    <p><strong>2024/04/08:</strong> Initial post.</p>
  </blockquote>
</div>
