---
layout: post
title: "KubeInit External access for OpenShift/OKD deployments with Libvirt"
author: "Carlos Camacho"
categories:
  - blog
tags:
- draft
- kubernetes
- kubeinit
- cloud
- okd
favorite: true
commentIssueId: 71
refimage: '/static/kubeinit/net/thumb.png'
---

In this post we will describe the basic network architecture used when OKD is
deployed using KubeInit in a KVM host.
Then we will describe how to extend the basic network configuration to provide
external access to the cluster services.

![](/static/kubeinit/net/thumb.png)

Here is the server current status:

```bash
[root@nyctea ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    100    0        0 eno1
10.19.41.0      0.0.0.0         255.255.255.0   U     100    0        0 eno1
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
```

```bash
[root@nyctea ~]# nmcli con show
NAME         UUID                                  TYPE      DEVICE
System eno1  162499bc-a6fa-45db-ba76-1b45f0be46e8  ethernet  eno1   
virbr0       4ba12c69-3a8b-42e8-a9dd-bc020fdc1a90  bridge    virbr0
eno2         e19725f2-84f5-4f71-b300-469ffc99fd99  ethernet  --     
enp6s0f0     7348301f-8cae-4ab1-9061-97d7a344699c  ethernet  --     
enp6s0f1     8a96c226-959a-4218-b9f7-c3ab6ee3d02b  ethernet  --    
```

As we can see we have two physical network interfaces (eno1, and eno2) for which
only one is actually connected.

And this is the actual standard network architecture for a usual deployment.

![](/static/kubeinit/net/arch01.png)

The default deployment will install a multi-master cluster, with one worker node (up to 10).
From the above figure is possible to see:

* All cluster nodes are connected to the 10.0.0.0/24 network. This will be the
cluster management network, and the one will use to access the nodes within the hypervisor.

* The 10.0.0.0/24 network is defined as a Virtual Network Switch implementing
both NAT and DCHP for any interface connected to the `kimgtnet0` network.

* All bootstrap, master, and worker nodes are installed with Fedora CoreOS as is
the required OS for OKD > 4.

* The services machine has installed CentOS 8 with BIND, HAProxy, and NFS.

* Using DHCP, we assign the following IP mapping based on the MAC address of each node (defined
 in the Ansible inventory).

```bash
 # Master
 okd-master-01 ansible_host=10.0.0.1 mac=52:54:00:aa:6c:b1
 okd-master-02 ansible_host=10.0.0.2 mac=52:54:00:59:0e:e4
 okd-master-03 ansible_host=10.0.0.3 mac=52:54:00:b4:39:45

 # Worker
 okd-worker-01 ansible_host=10.0.0.4 mac=52:54:00:61:22:5a
 okd-worker-02 ansible_host=10.0.0.5 mac=52:54:00:21:fd:fd
 okd-worker-03 ansible_host=10.0.0.6 mac=52:54:00:4c:0a:81
 okd-worker-04 ansible_host=10.0.0.7 mac=52:54:00:54:ff:ac
 okd-worker-05 ansible_host=10.0.0.8 mac=52:54:00:4a:6b:f6
 okd-worker-06 ansible_host=10.0.0.9 mac=52:54:00:40:22:52
 okd-worker-07 ansible_host=10.0.0.10 mac=52:54:00:6c:0a:03
 okd-worker-08 ansible_host=10.0.0.11 mac=52:54:00:0b:14:f8
 okd-worker-09 ansible_host=10.0.0.12 mac=52:54:00:f5:6e:e5
 okd-worker-10 ansible_host=10.0.0.13 mac=52:54:00:5c:26:4f

 # Service
 okd-service-01 ansible_host=10.0.0.100 mac=52:54:00:f2:46:a7

 # Bootstrap
 okd-bootstrap-01 ansible_host=10.0.0.200 mac=52:54:00:6e:4d:a3
```

> The previous deployment can be used for any purpose but it has one limitation,
> this limitation is that the endpoints do not have external access.
> This means that i.e. https://console-openshift-console.apps.watata.kubeinit.local
> can not be accessed from anywhere instead the hypervisor itself.

This post will show different network configurations to make the endpoints
accessible from an external interface.

## Requirements

* An additional IP address to be mapped to the services machine from an external location.
* Creating a network bridge to slave the interface we will use.

If a user has one extra IP (public or private) it will be enough to configure remote
access to the cluster endpoints.

As long as we have available an extra IP it does not matter how many physical interfaces we
have, as we can have multiple IP addresses configured using a single physical NIC.

This is the resulting network architecture to access remotely our freshly installed OKD cluster.

![](/static/kubeinit/net/arch02.png)

> Our development environment has only one network card connected,
> in this case after we create the main switch and slave the
> network device, it will lose the assigned IP automatically.
> Do not try this using a shell as you will get dropped.

To deploy this architecture please follow the next steps:

We create an initial bridge using the cockpit
losing the IP it will be recovered automatically
(don't try this from the CLI as you will lose access).

In this case,

Create a bridge called kiextbr0 connected to eno1:

![](/static/kubeinit/net/cockpit_00.PNG)
Click on: Networking -> Add Bridge

![](/static/kubeinit/net/cockpit_01.PNG)
Write: kiiextbr0 as the bridge name, and select
your network inferface.

![](/static/kubeinit/net/cockpit_02.PNG)
Check that the bridge is created correctly and has
the IP configured correctly.

As an example you can run these steps by the CLI
adjusting your interface and bridge names accordingly.

```bash
nmcli connection add ifname br0 type bridge con-name br0
nmcli connection add type bridge-slave ifname enp0s25 master br0
nmcli connection modify br0 bridge.stp no
nmcli connection delete enp0s25
nmcli connection up br0
```

>  _NOTE:_ If you have only one interface
> the connection will be dropped and you will.
> lose connectivity.

We check again the system status:

```bash
[root@nyctea ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    425    0        0 kilocbr0
10.19.41.0      0.0.0.0         255.255.255.0   U     425    0        0 kilocbr0
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0

[root@nyctea ~]# nmcli con show
NAME         UUID                                  TYPE      DEVICE  
kilocbr0      1c4d60a3-06a7-429f-a689-ffba5a49efbb  bridge    kilocbr0
System eno1  162499bc-a6fa-45db-ba76-1b45f0be46e8  ethernet  eno1    
virbr0       4ba12c69-3a8b-42e8-a9dd-bc020fdc1a90  bridge    virbr0  
eno2         e19725f2-84f5-4f71-b300-469ffc99fd99  ethernet  --      
eno3         65be9380-980b-4237-b27c-2479e8f8535d  ethernet  --      
eno4         9f5afe2d-6166-4197-a23f-e64c3b1b5ab2  ethernet  --      
enp6s0f0     7348301f-8cae-4ab1-9061-97d7a344699c  ethernet  --      
enp6s0f1     8a96c226-959a-4218-b9f7-c3ab6ee3d02b  ethernet  --    
```

These variables are defined in the playbook (the location of these variables will change)
but not their name.

![](/static/kubeinit/net/config_vars.PNG)

Now we deploy as usual KubeInit:

> Remember that you can execute this deployment command before
> creating the bridge with the CentOS cockpit, the bridge creation
> has no impact in how we deploy KubeInit.

```bash
ansible-playbook \
    -v \
    --user root \
    -i ./hosts/okd/inventory \
    --become \
    --become-user root \
    -e "{ \
      'kubeinit_bind_external_service_interface_enabled': 'true', \
      'kubeinit_bind_external_service_interface': { \
        'attached': 'kiextbr0', \
        'dev': 'eth1', \
        'ip': '10.19.41.157', \
        'gateway': '10.19.41.254', \
        'netmask': '255.255.255.0' \
      } \
    }" \
    ./playbooks/okd.yml
```

The meaning of the variables are:

* kubeinit_bind_external_service_interface_enabled: true - This will enable
the Ansible configuration of the external interface,
the BIND update, and the additional interface in the
service node.

* kubeinit_bind_external_service_interface.attached: kiextbr0 - This is the
virtual bridge where we will plug the `eth1` interface of the services machine.
The bridge `MUST` be created first and slaving the physical interface we will use.

* kubeinit_bind_external_service_interface.dev: eth1 - This is the name of the
external interface we will add to the services machine.

* kubeinit_bind_external_service_interface.ip: 10.19.41.157 - The external IP address
of the services machine.

* kubeinit_bind_external_service_interface.gateway: 10.19.41.254 - The gateway IP address
of the services machine.

* kubeinit_bind_external_service_interface.netmask: 255.255.255.0 - The network mask of the external
interface of the services machine.

Some of the very interesting changes here is BIND configuration,

![](/static/kubeinit/net/bind_views.png)

In this case we have an `internal` and `external` view that will behave differently depending
on where the requests are originated from.

If a DNS request is created trough the cluster's external interface, the reply will be
created based on the external view, in this case we only reply with the external HAProxy endpoints
related to the services node, thus, we will only reply with `10.19.41.157` as it is the only that needs
to be presented externally.

The next step is to configure your local DNS server to point to `10.19.41.157`

```bash
[ccamacho@localhost]$ cat /etc/resolv.conf
nameserver 10.19.41.157
nameserver 8.8.8.8
```

After that you should be able to access the cluster without any issue and use it for any purpose you have.

![](/static/kubeinit/net/dashboard.PNG)

If you like this post, please try the code, raise issues, and ask for more details, features or
anything that you feel interested in. Also it would be awesome if you become a stargazer.

This is the main project [repository](https://github.com/ccamacho/kubeinit).
