# setup_kubernetes_cluster_with_kubeadm
In this simple hands-on lab we will practice the process of building a multi-node Kubernetes cluster. We'll use three EC2 instances on AWS to setup the cluster consisting of one control plane node and two worker nodes. <br>

## Objective
To gain deep understanding of the various components that make up a kubernetes cluster <br>

## Kubernetes Architecture and Components
![image](https://github.com/user-attachments/assets/7ad98bb6-c269-4b1f-8c7f-fc4146ebeeea) <br>

- ETCD <br>
Etcd is a distributed reliable key-value store that is simple, secure and fast. Etcd stores information regarding the cluster such as the nodes, pods, configs, secrets, accounts, roles, bindings etc. <br>

- APIServer <br>
This is the primary management component in kubernetes. Kube-api server is the only component that interacts directly with the etcd datastore. The other components use the kube-apiserver. <br>

- Cntroller manager <br>
Manages various controllers in kubernetes. <br>
A controller is a process that continuously monitors the state of various components within the system and works towards bringing the whole system to the desired functioning state. <br>
  - Node controller: responsible for monitoring the status of the node and taking necessary actions to keep the application running. It does that through the kube-apiserver. <br>
  - Replication controller: responsible for monitoring the status of replica sets and ensuring that the desired number of pods are available at all times within the set. <br>
  - There are other types of controllers under the kube-controller manager e.g. deployment controller, namespace controller, job controller etc. <br>

- Scheduler <br>
Responsible for deciding which pod goes on which node depending on certain criteria. It doesn’t actually place the pods on the nodes, that’s the job of the kubelet. <br>

- Kubelet <br>
An agent that runs on each worker node, ensuring containers are running as defined. It manages communication between the node and the cluster control plane. <br>

- Kubeproxy <br>
Kubernetes component that runs on every node in the cluster and is responsible for managing network communication between pods, services, and external clients. <br>

- Container Runtime <br>
Software responsible for running containers by managing their lifecycle, including creation, execution, and deletion. Examples include Docker, containerd, and CRI-O. <br>



## What is Kubeadm 
Kubeadm is a tool used to setup kubernetes cluster. It simplifies the process of setting up kubernetes cluster components and follows best practices for cluster configuration.

## How Kubeadm works
When you initialise kubeadm, first it runs all the preflight checks to validate the system state and it downloads all the required cluster container images from ```registry.k8s.io```container registry. It then generates the required TLS certificates and stores it on the ```/etc/kubernetes/pki``` folder. Next, it generates all the kubeconfig files for the cluster component in ```/etc/kubernetes``` folder. It then starts the kubelet service and generates the static pod manifest for all the cluster components and saves it in the ```/etc/kubernetes/manifests``` folder. Next it starts all the control plane components from the static pod manifests. Then it installs CoreDNS and Kubeproxy components. Finally, it generates node bootstrap token which is used by the worker nodes to join the control plane.


# Implementation

## Setup Overview
- Deploy 3 EC2 instances and configure required security group rules.
- Install container runtime on all the instances. We'll be using Cri-O.
- Install Kubeadm, Kubelet and Kubectl on all the instances.
- Initialize Kubeadm control plane configuration on the master node.
- Join the worker node to the control plane.
- Install the calico network plugin to enable pod networking.
- Install Kubernetes metric server to enable pod and node metrics.
- Validate all cluster components and nodes
- Deploy sanple nginx app and validate the cluster

## Step1: Deploy EC2 instances and Configure Security groups
![image](https://github.com/user-attachments/assets/070767b8-9f08-4732-9ae8-91b85f8b21e1) <br>
![image](https://github.com/user-attachments/assets/4e76d109-987e-4ecb-9bad-48dd258d7055) <br>

## Step 2: Enable iptables bridged traffic on all the nodes
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

## Step 3: Disable swap on all the Nodes
```
sudo swapoff -a
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true
```
## Step 4: Install Cri-O Runtime on all the nodes
```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/Release.key | \
    sudo gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/ /" | \
    sudo tee /etc/apt/sources.list.d/cri-o.list

```
```
sudo apt-get update -y
sudo apt-get install -y cri-o
```
```
sudo systemctl daemon-reload
sudo systemctl enable crio --now
sudo systemctl start crio.service
```
## Step 5: Install crictl
```
VERSION="v1.30.0"
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz
```
## Step 6: Install kubeadm, kubelet and kubectl on all nodes
```
KUBERNETES_VERSION=1.30

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v$KUBERNETES_VERSION/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v$KUBERNETES_VERSION/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```
sudo apt-get update -y
```
```
sudo apt-get install -y kubelet kubeadm kubectl
```
## Step 7: Initialize Kubeadm control plane configuration on the master node to setup control plane
```
IPADDR=$(curl ifconfig.me && echo "")
NODENAME=$(hostname -s)
POD_CIDR="192.168.0.0/16"
```
```
sudo kubeadm init --control-plane-endpoint=$IPADDR  --apiserver-cert-extra-sans=$IPADDR  --pod-network-cidr=$POD_CIDR --node-name $NODENAME --ignore-preflight-errors Swap
```
![image](https://github.com/user-attachments/assets/d32bf3ab-65fe-4a06-a4de-c8ba0a4028bd) <br>
![image](https://github.com/user-attachments/assets/7fbe4e30-4cb2-419c-8462-a6e25834135e) <br>

Use the following commands from the output to create the kubeconfig in master so that you can use kubectl to interact with cluster API.

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



