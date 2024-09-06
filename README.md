# Kubernetes cluster No. 1

The cybros labs K8S cluster no. 1 is a Prague located main cluster.

## Network - added

This section provides a detailed explanation of the network configuration, including public IP addresses, CIDR ranges, and floating IPs used within the infrastructure.

This is new

- WAN (Port 9): `82.142.74.110`
  - This is the primary public IP address assigned to the WAN interface on port 9. It is used for inbound and outbound traffic to/from the network.
- WAN2 (Port 10): `82.142.91.12`
  - This is the secondary public IP address assigned to the WAN2 interface on port 10, often used for redundancy or load balancing.
- Network CIDR: `10.77.0.0/22` (`10.77.0.0-10.77.3.255`)

Floating IPs are virtual IP addresses that can be moved between different services or hosts within the network. They provide high availability and flexibility by allowing services to be reachable even if the underlying hardware changes.

- Floating IPs
  - `10.77.1.254`: This floating IP is assigned to the ETCD service, which is a key-value store used for storing configuration data and state information in distributed systems.
  - `10.77.1.253`: This floating IP is assigned to the Ingress controller, which manages external access to services within the Kubernetes cluster, typically via HTTP/HTTPS routes.

## Servers

| **HW**    | **CPU cores** | **RAM** | **Net** | **Name**        | **IP**     | **Notes**           | **Other**      |
| --------- | ------------- | ------- | ------- | --------------- | ---------- | ------------------- | -------------- |
| Dell R320 | 12            | 64GiB   | iDRAC   | ddczprgsv000010 | 10.77.0.10 |                     | root/tech.Bros |
|           |               |         | Server  | ddczprgc1n10    | 10.77.1.10 | K8S Node 1          |                |
| Dell R630 | 56            | 128GiB  | iDRAC   | ddczprgsv000011 | 10.77.0.12 |                     | root/tech.Bros |
|           |               |         | Server  | ddczprgc1n11    | 10.77.1.12 | K8S Node 2          |                |
| Dell R630 | 56            | 128GiB  | iDRAC   | ddczprgsv000012 | 10.77.0.12 |                     | root/tech.Bros |
|           |               |         | Server  | ddczprgc1n12    | 10.77.1.12 | K8S Node 3          |                |
| IBM       | 16            | 64GiB   | iDRAC   | ddczprgsv000013 | 10.77.0.13 |                     | root/tech.Bros |
|           |               |         | Server  | ddczprgc1n13    | 10.77.1.13 | K8S Node 4          |                |
| Dell R620 | 12            | 32GiB   | iDRAC   | ddczprgsv000031 | 10.77.0.31 |                     | root/tech.Bros |
|           |               |         | Server  | ddczprgs1n31    | 10.77.1.31 | CEPH Storage Node 1 |                |
| Dell R620 | 12            | 32GiB   | iDRAC   | ddczprgsv000032 | 10.77.0.32 |                     | root/tech.Bros |
|           |               |         | Server  | ddczprgs1n32    | 10.77.1.32 | CEPH Storage Node 2 |                |
| Dell R620 | 12            | 32GiB   | iDRAC   | ddczprgsv000033 | 10.77.0.33 |                     | root/tech.Bros |
|           |               |         | Server  | ddczprgs1n33    | 10.77.1.33 | CEPH Storage Node 3 |                |

## Operating Systems

All above servers use **Debian 12** LTS.

- Type: Basic install
- Additional services:

  - OpenSSH

- Credentials

  - RootP: master
  - U/P: master/master

- Main required packages:

  ```sh
  apt install -y open-iscsi sudo curl ifenslave mc lshw net-tools lm-sensors
  ```

> [!IMPORTANT]
> Do NOT install **ceph** package as it breaks the CEPH node!

- DNS settings:

  ```sh
  echo '# domain cybros.labs
  # search cybros.labs
  nameserver 10.77.0.1
  nameserver 1.1.1.1
  ' >/etc/resolv.conf
  ```

- Network setup for Dell R630 ddczprgc1n11, Dell R630 ddczprgc1n12, IBM ddczprgc1n13

  _Do not forget to adjust the IP address!_

  ```sh
  cat >/etc/network/interfaces <<EOF
  # This file describes the network interfaces available on your system
  # and how to activate them. For more information, see interfaces(5).

  source /etc/network/interfaces.d/*

  # The loopback network interface
  auto lo
  iface lo inet loopback

  auto bond0
  iface bond0 inet static
    address 10.77.1.11
    netmask 255.255.252.0
    network 10.77.0.0
    gateway 10.77.0.1
    bond-slaves eno1 eno2 eno3 eno4
    bond-mode 4
    bond-lacp-rate 1
    bond-miimon 100
    bond-xmit-hash-policy layer2+3

  auto eno1
  iface eno1 inet manual
    bond-master bond0

  auto eno2
  iface eno2 inet manual
    bond-master bond0

  auto eno3
  iface eno3 inet manual
    bond-master bond0

  auto eno4
  iface eno4 inet manual
    bond-master bond0
  EOF
  ```

- Network setup for Dell R320 ddczprgc1n10

  _Do not forget to adjust the IP address!_

  ```sh
  cat >/etc/network/interfaces <<EOF
  # This file describes the network interfaces available on your system
  # and how to activate them. For more information, see interfaces(5).

  source /etc/network/interfaces.d/*

  # The loopback network interface
  auto lo
  iface lo inet loopback

  auto bond0
  iface bond0 inet static
    address 10.77.1.10
    netmask 255.255.252.0
    network 10.77.0.0
    gateway 10.77.0.1
    bond-slaves enp8s0f0 enp8s0f1
    bond-mode 4
    bond-lacp-rate 1
    bond-miimon 100
    bond-xmit-hash-policy layer2+3

  auto enp8s0f0
  iface enp8s0f0 inet manual
    bond-master bond0

  auto enp8s0f1
  iface enp8s0f1 inet manual
    bond-master bond0
  EOF
  ```

- Network setup for Dell R620 ddczprgs1n3[1-3]

  _Do not forget to adjust the IP address!_

  ```sh
  cat >/etc/network/interfaces <<EOF
  # This file describes the network interfaces available on your system
  # and how to activate them. For more information, see interfaces(5).

  source /etc/network/interfaces.d/*

  # The loopback network interface
  auto lo
  iface lo inet loopback

  auto bond0
  iface bond0 inet static
    address 10.77.1.31
    netmask 255.255.252.0
    network 10.77.0.0
    gateway 10.77.0.1
    bond-slaves eno1 eno2 eno3 eno4
    bond-mode 4
    bond-lacp-rate 1
    bond-miimon 100
    bond-xmit-hash-policy layer2+3

  auto eno1
  iface eno1 inet manual
    bond-master bond0

  auto eno2
  iface eno2 inet manual
    bond-master bond0

  auto eno3
  iface eno3 inet manual
    bond-master bond0

  auto eno4
  iface eno4 inet manual
    bond-master bond0
  EOF
  ```

### Addons for CEPH servers

No addons are needed.

### Addos for Kubernetes servers

```sh
apt install -y podman cephadm
```

## Kubernetes nodes

When all above prerequisites are met, Kubernetes can be deployed. We use [RKE2](https://docs.rke2.io) distribution.

- Master nodes

  ```sh
  curl -sfL https://get.rke2.io | INSTALL_RKE2_VERSION=v1.28.11+rke2r1 sh -
  ```

- Agent nodes

  ```sh
  curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE=agent INSTALL_RKE2_VERSION=v1.28.11+rke2r1 sh -
  ```

kube-vip is deployed as _DemonSet_ on master nodes (using node affinity, nodes with label `node-role.kubernetes.io/master=true`) in ARP mode, see [docs](https://kube-vip.io/docs/installation/daemonset/).

### HA control plane

To run the cluster control plane HA setup, we are using the [kube-vip](https://kube-vip.io/) (as mentioned before) with floating (virtual) IP address `10.77.1.254` (configured on Ubiquiti router).

For more information on kube-vip and HA control plane, see [kube-vip docs](https://kube-vip.io/docs/installation/daemonset/) on the topic.

### HA Ingress traffic

> [!IMPORTANT]
> Before any packet reaches our infrastructure at Smíchov Bussiness Park, traffic is routed through Cloudflare Load Balancer with DNS records in Proxy mode (all traffic goes to Cloudflare infrastructure first and then to our (origin) servers).
> Target origin servers (IPs) are our two public IP addresses, see [Network](#network).

Cloudflare Load Balancer: `lb.cybroslabs.com`. All of our DNS names pointing to our infrastructure are CNAME records pointing to `lb.cybroslabs.com`.

> [!NOTE]
> All incoming HTTP and HTTPS traffic (ports `80` and `443`, any other port is not routed to this IP!) is being routed from Ubiquiti to the floating IP `10.77.1.253`.

The external traffic and all DNS names point to our public IP address, see [Network](#network).

#### Ingress controller

For Ingress Controller, we are using Kubernetes default Ingress Controller: [Ingress NGINX](https://kubernetes.github.io/ingress-nginx/).

The Ingress NGINX is deployed in _DemonSet_ mode, with node affinity set to master / control plane nodes only.

#### Highly available ingress controller

To run our cluster with high availability in mind, we are using [kube-vip](https://kube-vip.io/).

Ingress Controller high availability is achieved with kube-vip backed _Service_, `type=LoadBalancer` and [`loadBalancerClass=kube-vip.io/kube-vip-class`](https://kube-vip.io/docs/usage/kubernetes-services/#kubernetes-loadbalancer-class-kubernetes-v124-kube-vip-v050) and [`externalTrafficPolicy=Local`](https://kube-vip.io/docs/usage/kubernetes-services/#external-traffic-policy-kube-vip-v050) [`externalTrafficPolicy` Kubernetes docs](https://kubernetes.io/docs/concepts/services-networking/service-traffic-policy/)).

This setup ensures that only one node has lease to be the leader (has the floating IP assigned) and is routing the traffic to all Services and Pods across the cluster.

The floating (virtual) IP is `10.77.1.253` (configured on Ubiquiti router).

## CEPH storage nodes

When all above prerequisites are met, CEPH cluster can be deployed. We use **cephadm** to deploy the CEPH.

> [!IMPORTANT]
> Do NOT install **ceph** package as it breaks the CEPH node!

[Manual](https://docs.ceph.com/en/latest/cephadm/install/)

All CEPH nodes must be resolvable by name, Dell R620 ddczprgs1n3[1-3]

```sh
echo '
10.77.1.31 ddczprgs1n31
10.77.1.32 ddczprgs1n32
10.77.1.33 ddczprgs1n33
' >>/etc/hosts
```

During Ceph installation of the 2nd and more nodes, the root user is being used. There is public key which is added using `ssh-copy-id` distributes the key. To make it working password auth for root can be temporarily enabled:

- `vi /etc/ssh/sshd_config`: **PermitRootLogin yes**
- `systemctl restart ssh`
- Disable after the key has been set! See below how to add 2nd and other nodes.

### Bootstrap 1st node

The first node has been bootstrapped using cephadm, more [here](https://docs.ceph.com/en/latest/cephadm/install/#bootstrap-a-new-cluster).

```sh
cephadm bootstrap --mon-ip 10.77.1.31
```

Ceph Dashboard is now available at:

- **URL**: https://10.77.1.31:8443/
- **User**: admin
- **Password**: tech.Bros

### More nodes to add

Here is an example how to add second node. Other nodes were added the same way, just change hostname and IP. Be ware that ceph command directly does not and won't work. Always use `cephadm shell -- ceph ...` command to execute ceph.
The **ceph.pub** contains SSH public key and it must be put into authorized_keys in the **root** user. The cept itself installs all the services automatically.

```sh
cephadm shell -- ssh-copy-id -f -i /etc/ceph/ceph.pub root@10.77.1.32
cephadm shell -- ceph orch host add ddczprgs1n32 10.77.1.32 --labels _admin
```

### Verify SSD vs. HDD

For RAID devices, Ceph is not able to detect SSD drives well and marks them as HDD. Use `ceph osd tree` to see the list of OSDs. To mark SSD drives it's needed to remove and add mark:

```sh
ceph osd crush rm-device-class osd.<N>
ceph osd crush set-device-class ssd osd.<N>
```

### Status validation

```sh
cephadm shell -- ceph -s
```

### Auth for Kubernetes plugins

- Get auth-key for **kubernetes** account:

  ```sh
  ceph auth get-or-create client.kubernetes mon 'profile rbd' osd 'profile rbd pool=kubernetes' mgr 'profile rbd pool=kubernetes'
  ```

- Get auth-key for **admin** account:
  ```sh
  ceph auth get-key client.admin
  ```

## Issue mitigation

> [!IMPORTANT]
>
> **Issue**: failed to create fsnotify watcher: too many open files
> **Solution**: Run on all Kubernetes nodes following command. Do it during node installation!
>
> ```sh
> echo "fs.inotify.max_user_instances=2048" >>/etc/sysctl.conf
> sysctl -w fs.inotify.max_user_instances=2048
> ```

> [!NOTE]
>
> **Issue**: The network connection breaks randomly.
> **Solution**: Do not use Ubuntu 24, Kernel 6.8. Severe network issues have been observed with this kernel. Use Debian instead.

## Hardware Monitoring

For HW monitoring purposes, we are using IPMI exporter (see argocd repository). It scrapes list of servers using IPMI tools. To enable this feature on the HW server following setup is needed.

```sh
apt install ipmitool freeipmi
ipmitool user set name 4 ipmi_exporter
ipmitool user set password 4 DicNu2HovIMKESo2
ipmitool user priv 4 2 1
ipmitool channel setaccess 1 4 callin=off ipmi=on link=off privilege=2
ipmitool user enable 4
```

The ipmitools configures underlaying HW (iDRAC) by setting ipmi_exporter user. This can be done via iDRAC instead. You can validate the IPMI by running `ipmitool -I lanplus -H 10.77.0.10 -U ipmi_exporter` towards given iDRAC IP.

> [!NOTE]
> The IPMI over LAN must be enabled in iDRAC!

## Glossary

- Machine: physical hardware, running an operating system
- Node: A machine which is a member of Kubernetes cluster
- Master (control plane) node: machine running Kubernetes Control plane (formerly master) components (etcd, scheduler, kubeapi, controller-manager)
- Worker node (or just node): node not running control plane components, only the basic ones needed (kubelet, kube-proxy, container runtime)
- _Pod_: Smallest deployabled unit in Kubernetes, describe single instance of an application running, can contain more than one container
- _DemonSet_: Kubernetes primitive describing containers running on each node of cluster
- Node affinity: forcing _Pods_ to run on selected nodes (nodes that are allowed by a condition)
- _Service_: Kubernetes primitive exposing in-cluster or even outside (with `type=LoadBalancer` or `type=NodePort`) a network interface (IP and port)
- _Ingress_: Kubernetes primitive describing an HTTP endpoint exposed from the cluster to outside world
- _Ingress Controller_: Kubernetes component (not provided by default), which is responsible for handling and provisioning a backend/proxy for any _Ingress_ resources and handling the incoming traffic and then routing it to corresponding _Services_

## History

### Obsolete - Hardware

Network description:

| **Hostname** | **Device**     | **Type**       | **IP**    | **Notes**   |
| ------------ | -------------- | -------------- | --------- | ----------- |
| ddczprggw01  | UDM SE         | Gateway        | 10.77.0.1 | via ui.com  |
| ddczprgsw01  | UDM-PRO-24     | Switch         | 10.77.0.2 | via ui.com  |
| ddczprgpw01  | UDM-PRO-24 POE | Switch         | 10.77.0.3 | via ui.com  |
| N/A          | UPS            | Smart-UPS 2200 | 10.77.0.5 | HTTPS on IP |

### Obsolete - Storage

- CEPH U/P: admin/tech.Bros

| **Node**     | **Type** | **Product**               | **Capacity**     | **Notes**                                     |
| ------------ | -------- | ------------------------- | ---------------- | --------------------------------------------- |
| ddczprgc1n10 | RAID 1   | PERC H710                 | 278GiB (299GB)   | VER=3.13 S/N=00994b040826f1b02900888e0320844a |
|              | RAID     | PERC H710                 | 2233GiB (2398GB) | VER=3.13 S/N=004ba6aa0942f1b02900888e0320844a |
|              | SAS      | Seagate 10k 6Gbps SAS     | 278.875          | ST300MM0006                                   |
|              | SAS      | Seagate 10k 6Gbps SAS     | 278.875          | ST300MM0006                                   |
|              | SAS      | Seagate 10k 6Gbps SAS     | 558.375          | ST600MM0006                                   |
|              | SAS      | Seagate 10k 6Gbps SAS     | 558.375          | ST600MM0006                                   |
|              | SAS      | Seagate 10k 6Gbps SAS     | 558.375          | ST600MM0006                                   |
|              | SAS      | Seagate 10k 6Gbps SAS     | 558.375          | ST600MM0006                                   |
|              | SAS      | Seagate 10k 6Gbps SAS     | 558.375          | ST600MM0006                                   |
|              | SAS      | Seagate 10k 6Gbps SAS     | 558.375          | ST600MM0006                                   |
| ddczprgc1n11 | RAID 1   | PERC H730P Mini           | 475GiB (510GB)   | VER=4.30 S/N=00194d67081a9e3628003f874b708741 |
|              | RAID 10  | PERC H730P Mini           | 893GiB (959GB)   | VER=4.30 S/N=0045bc1a0a806b3728003f874b708741 |
|              | SSD      | Samsung SSD 860           | 237.88 GB        | REV=RVM02B6Q S/N=S42VNX0N702714D              |
|              | SSD      | Samsung SSD 860           | 237.88 GB        | REV=RVM02B6Q S/N=S42VNX0N702731X              |
|              | SSD      | Samsung SSD 860           | 237.88 GB        | REV=RVM02B6Q S/N=S42VNX0N702716F              |
|              | SSD      | Samsung SSD 860           | 237.88 GB        | REV=RVM02B6Q S/N=S42VNX0N702748W              |
|              | SSD      | KINGSTON SEDC500          | 446.63 GB        | REV=SCEKJ2.7 S/N=50026B72827ED8E3             |
|              | SSD      | KINGSTON SEDC500          | 446.63 GB        | REV=SCEKJ2.7 S/N=50026B72827ED8CB             |
|              | SSD      | KINGSTON SEDC500          | 446.63 GB        | REV=SCEKJ2.7 S/N=50026B72827ED8DF             |
|              | SSD      | KINGSTON SEDC500          | 446.63 GB        | REV=SCEKJ2.7 S/N=50026B72827ED847             |
| ddczprgc1n12 | RAID 1   | PERC H730P Mini           | 475GiB (510GB)   | VER=4.30 S/N=00194d67081a9e3628003f874b708741 |
| ddczprgc1n13 | RAID 1   | ServeRAID M5210           | 464GiB (498GB)   | VER=4.22 S/N=00d1837f35524eaf1ca00b150ab00506 |
|              | SATA     | Express IBM 7.2k 6Gbps NL | 500GB            | ST9500620NS 00AJ137 00AJ140IBM                |
|              | SATA     | Express IBM 7.2k 6Gbps NL | 500GB            | ST9500620NS 00AJ137 00AJ140IBM                |
| ddczprgs1n1  | RAID 1   |                           |                  |                                               |
| ddczprgs1n2  | RAID 1   |                           |                  |                                               |
| ddczprgs1n3  | RAID 1   |                           |                  |                                               |

#### Obsolete - Per machine setup

- _OPTIONAL_: Add environment variables to `/etc/environment`

  ```sh
  PATH="...:/var/lib/rancher/rke2/bin"
  KUBECONFIG="/etc/rancher/rke2/rke2.yaml"
  ```

- Ubuntu 22.04 Kernel module for PostgreSQL (and possible other programs). Required, because we are running Kernel `5.15.x`

  ```sh
  echo "GRUB_CMDLINE_LINUX=systemd.unified_cgroup_hierarchy=false" | sudo tee /etc/default/grub.d/cgroup.cfg
  sudo update-grub
  reboot
  ```

- Ubuntu 24.04 Kernel module for CEPH

  ```sh
  cat <<EOF >/etc/modules-load.d/kubeadm.conf
  br_netfilter
  rbd
  EOF
  cat <<EOF >/etc/sysctl.d/kubeadm.conf
  net.ipv4.ip_forward = 1
  EOF
  ```
