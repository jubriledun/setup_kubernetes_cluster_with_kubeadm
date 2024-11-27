# setup_kubernetes_cluster_with_kubeadm
In this simple hands-on lab we will practice the process of building a multi-node Kubernetes cluster. We'll use three EC2 instances on AWS to setup the cluster consisting of one master node and two worker nodes. <br>

### Objective
To gain deep understanding of the various components that make up a kubernetes cluster <br>

### Kubernetes Architecture and Components of a Kubernetes Cluster
![image](https://github.com/user-attachments/assets/cd4b735a-3b92-42e1-97c8-427ae76e80f2) <br>



### What is Kubeadm 
Kubeadm is a tool used to setup kubernetes cluster. It simplifies the process of setting up kubernetes cluster components and follows best practices for cluster configuration. <br>


A Kubernetes Cluster needs a Control plane node and at least one Worker node. for this lab, we have three Linux servers. One of the servers will function as the Control plane server(node), and two of the servers will be our Worker node
