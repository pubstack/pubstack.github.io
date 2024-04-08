---
layout: post
title: "Sharing your GPU in the cloud"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - draft
  - cloud
  - kubernetes
  - engineering
  - openshift
hidden: true
favorite: true
commentIssueId: 93
refimage: '/static/gpu_post/cloud.jpg'
---

In this post we will introduce different methods for sharing GPU resources
across workloads in K8s clusters.

GPU sharing refers to the practice of allowing multiple users or processes
to access and utilize the resources of a single Graphics Processing Unit (GPU) concurrently.
This approach can be beneficial in scenarios where there are multiple workloads
that can utilize GPU resources intermittently or simultaneously, such as in cloud
computing environments, data centers, or research institutions.

* Time-slices: Time-slices refers to a method of GPU sharing where the GPU resources
are allocated to different users or processes in time slices or intervals.
<div style="float: right; width: 230px; background: white;"><img src="/static/gpu_post/slice.png" alt="" style="border:15px solid #FFF"></div>
Each user or process is given access to the GPU for a certain period before it is switched to another
user or process. Time-sharing can help in maximizing GPU utilization and accommodating
multiple users with varying resource requirements.

* Multi-Instance GPU (MIG): MIG is a feature introduced by NVIDIA that allows a single
physical GPU to be partitioned into multiple instances, each with its own dedicated
compute resources, memory, and performance profiles. This enables multiple users or
workloads to run concurrently on the same physical GPU with isolation and performance guarantees.

* Multi-Process Service (MPS): MPS is a feature provided by NVIDIA GPUs that enables
multiple CUDA applications to share a single GPU concurrently. It allows the GPU to
switch between different CUDA contexts rapidly, enabling better utilization of GPU
resources across multiple processes.

Overall, these techniques aim to improve the efficiency of GPU utilization and
accommodate the diverse needs of users or processes sharing the GPU resources.

![](/static/gpu_post/gpu_lego.png)

# Enabling Time-slicing

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
            replicas: 8
EOF
```

With the resource created we need to patch the initial ClusterPolicy
from the GPU operator 'gpu-cluster-policy'.

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
#  devicePlugin:
#    config:
#      default: ''
#      name: time-slicing-config
#    enabled: true
#    mps:
#      root: /run/nvidia/mps
```

So we fetch the configuration from the previously created config map `time-slicing-config`.

To make sure the NFD operator runs we label the node stating that the `device-plugin.config` should point to `NVIDIA-A100-PCIE-40GB`. And we label the node we want to apply the configuration

```bash
oc label --overwrite node perf-intel-6.perf.eng.bos2.dc.redhat.com nvidia.com/device-plugin.config=NVIDIA-A100-PCIE-40GB
```

After a few minutes we can change that the NFD operator reconfigured the node to use time-slicing

```bash
oc get node --selector=nvidia.com/gpu.product=NVIDIA-A100-PCIE-40GB-SHARED -o json | jq '.items[0].status.capacity'
```

```bash
# {
#   "cpu": "128",
#   "ephemeral-storage": "3123565732Ki",
#   "hugepages-1Gi": "0",
#   "hugepages-2Mi": "0",
#   "memory": "527845520Ki",
#   "nvidia.com/gpu": "8",
#   "pods": "250"
# }
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
(dcgmproftester12)[https://docs.nvidia.com/datacenter/dcgm/latest/user-guide/feature-overview.html#cuda-test-generator-dcgmproftester].
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

After `dcgmproftester12` finishes, the pods logs show:

```bash
Skipping CreateDcgmGroups() since DCGM validation is disabled
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.78e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.96e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.97e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.89e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.92e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.94e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.94e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.90e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.92e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.93e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.93e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.90e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.92e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.92e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.91e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.90e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.90e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.91e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.91e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.90e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.90e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.90e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.89e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.89e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.90e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.90e+05 gflops).
Worker 0:0 [1004]: TensorEngineActive: generated ???, dcgm 0 (1.89e+05 gflops).
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

# Enabling Multi-Instance GPU (MIG)

Similarly as time-slicing the configuration needs to be adjusted using both configmaps
and updating the cluster policy to the changes are applied correctly. The labeling
step done in time-slicing needs to be executed only once.



#
# Enabling MPS
#

# We go through a similar process as enabling timesharing

# We create a config map with the MPS configuration

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
        renameByDefault: true
        resources:
        - name: nvidia.com/gpu
          # replicas: 49 # This will make it break
          replicas: 10
EOF

# And we path the cluster policy to apply the configuration from the new config map

# Now we patch the CR
oc patch clusterpolicy \
    gpu-cluster-policy \
    -n nvidia-gpu-operator \
    --type merge \
    -p '{"spec": {"devicePlugin": {"config": {"name": "mps-enable-config"}}}}'

# With the cluster policy patched we dont have to relabel the node because it was done in a previous step
# To trigger the NFD gpu-feature-discovery-XXX POD YOU NEED TO MAKE SURE THE CLUSTER POLICY IS PATCHED.

# Now the product name should change from -SHARED to the original name

oc get node --selector=nvidia.com/gpu.product=NVIDIA-A100-PCIE-40GB -o json | jq '.items[0].status.capacity'

# And we query the gpu labels

kubectl describe node perf-intel-6.perf.eng.bos2.dc.redhat.com | awk '/^Labels:/,/^Taints:/' | grep nvidia.com

# And these should be available

# nvidia.com/gpu.sharing-strategy=mps
# nvidia.com/mps.capable=true

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



Now we moved from

                    nvidia.com/gpu=1
to
                    nvidia.com/gpu.count=1

cat << EOF | kubectl create -f -
---
apiVersion: v1
kind: Pod
metadata:
  generateName: command-nvidia-smi-
  # name: command-nvidia-smi
spec:
  restartPolicy: Never
  containers:
    - name: cuda-container
      image: nvcr.io/nvidia/cuda:12.1.0-base-ubi8
      command: ["/bin/sh","-c"]
      args: ["nvidia-smi"]
      resources:
        limits:
          nvidia.com/gpu.shared: "1"
  # nodeSelector:
  #   nvidia.com/gpu.product: NVIDIA-A100-PCIE-40GB
EOF



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
            - while true; do /usr/bin/dcgmproftester12 --no-dcgm-validation -t 1004 -d 300; sleep 30; done
         resources:
           limits:
             # To check gpu vs gpu.shared
             nvidia.com/gpu.shared: "1"
         securityContext:
           capabilities:
             add: ["SYS_ADMIN"]
EOF



docker pull nvcr.io/nvidia/samples:dcgmproftester-2.0.10-cuda11.0-ubuntu18.04

docker pull nvidia/samples:dcgmproftester-2.0.10-cuda11.0-ubuntu18.04
docker tag nvidia/samples:dcgmproftester-2.0.10-cuda11.0-ubuntu18.04 quay.io/ccamacho/nvidia/samples/dcgmproftester:2.0.10-cuda11.0-ubuntu18.04



cat << EOF | kubectl apply -f -
---
apiVersion: v1
kind: Pod
metadata:
  generateName: cuda-vectoradd-
spec:
  restartPolicy: OnFailure
  containers:
  - name: vectoradd
    image: nvcr.io/nvidia/k8s/cuda-sample:vectoradd-cuda11.7.1-ubi8
    resources:
      limits:
        nvidia.com/gpu.shared: "1"
  nodeSelector:
    nvidia.com/gpu.product: NVIDIA-A100-PCIE-40GB
EOF






## Update log:

<div style="font-size:10px">
  <blockquote>
    <p><strong>2023/05/26:</strong> Initial version.</p>
  </blockquote>
</div>
