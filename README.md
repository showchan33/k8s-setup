# About k8s-setup

K8s-setup creates a Kubernetes cluster on a Linux machine. Ansible and kubeadm are used to create Kubernetes clusters.<br>
Ansible Playbook is typically run from a different machine than the one on which Kubernetes cluster is created. The figure below illustrates this.

![](./doc/k8s-setup.png)

# Requirement

## On the machine running Ansible

* OS
    * Linux (Checked on Ubuntu 20.04)
* Tool
    * Ansible

## On each node in a Kubernetes cluster

* OS
    * Ubuntu (Checked on Ubuntu 20.04)
* Tool
    * Python3

# Preparation

## Prepare public and private keys for SSH connection

SSH to each machine on which you will create Kubernetes to perform setup. Prepare a pair of public and private keys for this purpose.<br>
Add the public key to ``/home/[username]/.ssh/authorized_keys`` on each node. After this you should be able to connect via SSH with the following command.

```
ssh -i [filename of private key] [username]@[hostname]
```

## Prepare configuration files
Create a copy of the ``*.sample`` file without the trailing ``.sample``, as follows.

```
cp files/sshkey.sample files/sshkey
cp group_vars/all.yml.sample group_vars/all.yml
cp hosts.sample hosts
cp host_vars/control-plane1.yml.sample host_vars/control-plane1.yml
cp host_vars/worker-node1.yml.sample host_vars/worker-node1.yml
(Copy the number of worker nodes to be created)
```

Edit the contents of each config file according to your environment.

### files/sshkey

Stores the private key used for SSH connections. It will be in the following format.
```
-----BEGIN OPENSSH PRIVATE KEY-----
(BASE64-encoded text with the private key)
-----END OPENSSH PRIVATE KEY-----
```

### group_vars/all.yml

Define variables that are common to all hosts (i.e., nodes).
```
ansible_user: user                              # Fill in the user name of the remote host.
ansible_ssh_private_key_file: "./files/sshkey"  # Specify the path to the private key file.

file_kubeadm_join_command: "./kubeadm_join_command.tmp" # Editing is not necessary.
```

### hosts

Group hosts by controla plane and worker node and define a name for each host.<br>
Variables for each group of hosts are set in ``host_vars/[hostname].yml`` described below.
```
[control-plane] # Host group of control plane
control-plane1

[worker-node]   # Host group of worker node 
worker-node1
(Add worker nodes to be created, if necessary)
```

### host_vars/control-plane1.yml

Define variables to be used in the control plane.<br>

```
ansible_host: control-plane1.example.com
# node_ip: 10.0.0.4
# kubeadm:
#   pod_network_cidr: 192.168.0.0/16
#   service_cidr: 10.96.0.0/12
#   apiserver_advertise_address: "{{ node_ip }}"
```

* Required
    * ansible_host : Host name for SSH connection. IP address can also be set.
* Optional
    * node_ip : IP address to connect to each Kubernetes node. Mainly used when you want to use an IP address different from ``ansible_host``. For example, a node's private IP, etc.
    * kubeadm section : Indicate if you want the CIDR of a Pod or Service to be different from the default value.

### host_vars/worker-node1.yml

Define variables to be used in the worker node.<br>

```
ansible_host: worker-node1.example.com
# node_ip: 10.0.0.5
```

* Required
    * ansible_host : Host name for SSH connection. IP address can also be set.
* Optional
    * node_ip : IP address to connect to each Kubernetes node.

# Creating a Kuberenetes cluster

## Building a control plane

Execute the following command.
```
ansible-playbook site_control-plane.yml
```

## Building a control plane

After building the control plane, execute the following command.
```
ansible-playbook site_worker.yml
```

# Notes and Constraints

* containerd is used as container runtime.
* CNI construction cannot be performed with this tool.
    * For your reference, the author wrote how to install Flannel on [Qiita](https://qiita.com/showchan33/items/02e4a5f02b08c08d7813#4-cni%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB). Please understand that this article is written in Japanese.
* The version of Kubernetes that was tested is as follows.

```
$ kubectl version
Client Version: v1.29.6
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.29.6

$ kubelet --version
Kubernetes v1.29.6

$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"29", GitVersion:"v1.29.6", GitCommit:"062798d53d83265b9e05f14d85198f74362adaca", GitTreeState:"clean", BuildDate:"2024-06-11T20:22:13Z", GoVersion:"go1.21.11", Compiler:"gc", Platform:"linux/amd64"}
```

# Author
showchan33

# License
"k8s-setup" is under [GPL license](https://www.gnu.org/licenses/licenses.en.html).
 
