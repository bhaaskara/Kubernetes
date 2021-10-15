# Kubernetes Basics  
## What is Kubernetes
Kubernetes is a container archestration tool.
is portable,extensible,and open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation.

The name Kubernetes originates from Greek, meaning helmsman or pilot.   
**K8s** as an abbreviation results from counting the eight letters between the "K" and the "s".  

Google open-sourced the Kubernetes project in 2014. Kubernetes combines over 15 years of Google's experience running production workloads at scale with best-of-breed ideas and practices from the community.

## K8s Capacity
At V1.19, K8s supports clusters with upto 5000 nodes.  
More specifically it supports configuration that meets all of the following criteria.  
Object | Limit
-----  | -----
Nodes | 5000
Total Pods | 150000
Total Containers | 300000
Pods per node | 100

# K8s Architecture
K8s follows master and worker node architecture, master node manages the worker nodes and the Pods in the cluster.
Master node (also referred as control plane) make global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events, while worker nodes host the Pods that are the components of the application workload.

![image](https://user-images.githubusercontent.com/41310048/137490691-8e19d883-b057-4b2e-94da-acd8a3e60dfe.png)
![image](https://user-images.githubusercontent.com/41310048/137490773-642c96de-bda9-4462-9554-70b2363bf428.png)

## K8S cluster components
When you deploy Kubernetes, you get a cluster.  
A Kubernetes cluster consists of components that represent the control plane and set of worker machines, called nodes, that run containerized applications.  
Every cluster has at least one worker node.

**Control plane**
The control plane manages the worker nodes and the Pods in the cluster. 
In production environments, the control plane usually runs across multiple computers and a cluster usually runs multiple nodes, providing fault-tolerance and high availability.
**Worker nodes**
The worker node(s) host the Pods that are the components of the application workload.

### Control plane components
The control plane's components make global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events (for example, starting up a new pod when a deployment's replicas field is unsatisfied).

Control plane components can be run on any machine in the cluster. However, for simplicity, set up scripts typically start all control plane components on the same machine, and do not run user containers on this machine.

Control plane setup that runs across multiple VMs.
[K8s HA](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/) 
