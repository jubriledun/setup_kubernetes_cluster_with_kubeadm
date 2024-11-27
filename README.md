# setup_kubernetes_cluster_with_kubeadm
In this simple hands-on lab we will practice the process of building a multi-node Kubernetes cluster. We'll use three EC2 instances on AWS to setup the cluster consisting of one control plane node and two worker nodes. <br>

### Objective
To gain deep understanding of the various components that make up a kubernetes cluster <br>

### Kubernetes Architecture and Components
![image](https://github.com/user-attachments/assets/cd4b735a-3b92-42e1-97c8-427ae76e80f2) <br>

ETCD <br>
Etcd is a distributed reliable key-value store that is simple, secure and fast. Etcd stores information regarding the cluster such as the nodes, pods, configs, secrets, accounts, roles, bindings etc. <br>

Kube-APIServer <br>
This is the primary management component in kubernetes. Kube-api server is the only component that interacts directly with the etcd datastore. The other components use the kube-apiserver. <br>

Kube-controller manager <br>
Manages various controllers in kubernetes. <br>
A controller is a process that continuously monitors the state of various components within the system and works towards bringing the whole system to the desired functioning state. <br>
- Node controller: responsible for monitoring the status of the node and taking necessary actions to keep the application running. It does that through the kube-apiserver. <br>
- Replication controller: responsible for monitoring the status of replica sets and ensuring that the desired number of pods are available at all times within the set. <br>
- There are other types of controllers under the kube-controller manager e.g. deployment controller, namespace controller, job controller etc. <br>

Kube Scheduler <br>
Responsible for deciding which pod goes on which node depending on certain criteria. It doesn’t actually place the pods on the nodes, that’s the job of the kubelet. <br>

Kubelet <br>
An agent that runs on each worker node, ensuring containers are running as defined. It manages communication between the node and the cluster control plane. <br>

Kube-proxy <br>
Kubernetes component that runs on every node in the cluster and is responsible for managing network communication between pods, services, and external clients. <br>



### What is Kubeadm 
Kubeadm is a tool used to setup kubernetes cluster. It simplifies the process of setting up kubernetes cluster components and follows best practices for cluster configuration. <br>


A Kubernetes Cluster needs a Control plane node and at least one Worker node. for this lab, we have three Linux servers. One of the servers will function as the Control plane server(node), and two of the servers will be our Worker node
