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
Kubeadm is a tool used to setup kubernetes cluster. It simplifies the process of setting up kubernetes cluster components and follows best practices for cluster configuration. <br>
## How Kubeadm works
When you initialise kubeadm, first it runs all the preflight checks to validate the system state and it downloads all the required cluster container images from registry.k8s.io container registry. <br> 
It then generates the required TLS certificates and stores it on the /etc/kubernetes/pki folder. <br>
Next, it generates all the kubeconfig files for the cluster component in /etc/kubernetes folder. <br>
It then starts the kubelet service and generates the static pod manifest for all the cluster components and saves it in the /etc/kubernetes/manifests folder. <br>
Next it starts all the control plane components from the static pod manifests. <br> 
Then it installs CoreDNS and Kubeproxy components. <br> 
Finally, it generates node bootstrap token which is used by the worker nodes to join the control plane.


# Implementation

## Setup Overview
- Deploy 3 EC2 instances and configure required security group rules.
- Install container runtime on all the instances. We'll be using Crio.
- Install Kubeadm, Kubelet and Kubectl on all the instances.
- Initiate Kubeadm control plane configuration on the master node.
- Join the worker node to the control plane.
- Install the calico network plugin to enable pod networking.
- Install Kubernetes metric server to enable pod and node metrics.
- Validate all cluster components and nodes
- Deploy sanple nginx app and validate the cluster

## Create the EC2 instances

