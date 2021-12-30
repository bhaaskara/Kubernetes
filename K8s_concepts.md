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

1. Kube API Server
2. etcd
3. Kube-scheduler
4. Kube-control-manager
5. Cloud-control-manager

> Control plane components can be run on any machine in the cluster. However, for simplicity, set up scripts typically start all control plane components on the same machine, and do not run user containers on this machine.

Control plane setup that runs across multiple VMs.
[K8s HA](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/) 



#### Kube API Server
The core of Kubernetes' control plane is the API server and the HTTP API that it exposes.  
Users, the different parts of your cluster, and external components all communicate with one another through the API server.  
The API server is the front end for the Kubernetes control plane.  
kube-apiserver is designed to scale horizontally—that is, it scales by deploying more instances.  
You can run several instances of kube-apiserver and balance traffic between those instances.

Kubectl command’s job is to connect to kube-apiserver and communicate with it using the Kubernetes API.  
kube-apiserver also authenticates incoming requests, determines whether they are authorized and valid, and manages admission control.  
But it’s not just kubectl that talks with kube-apiserver. In fact, any query or change to the cluster’s state must be addressed to the kube-apiserver.

#### etcd
etcd is the cluster’s database.  
Its job is to reliably store the state of the cluster. This includes all the cluster configuration data; and more dynamic information such as what nodes are part of the cluster, what Pods should be running, and where they should be running.  
You never interact directly with etcd; instead, kube-apiserver interacts with the database on behalf of the rest of the system.

#### Kube-scheduler
Control plane component that watches for newly created Pods with no assigned node, and selects a node for them to run on.  
But it doesn’t do the work of actually launching Pods on Nodes. Instead, whenever it discovers a Pod object that doesn’t yet have an assignment to a node, it
chooses a node and simply writes the name of that node into the Pod object. 
Factors taken into account for scheduling decisions include:  
 - individual and collective resource requirements,  
 - hardware/software/policy constraints  
 - affinity (which cause groups of pods to prefer running on the same node)  
 - anti-affinity (which ensure that pods do not run on the same node) specifications  
 - data locality  
 - inter-workload interference  
 - deadlines

#### Kube-controller-manager
Control Plane component that runs controller processes.
kube-controller-manager has a broader job. It continuously monitors the state of a cluster through Kube-APIserver. Whenever the current state of the cluster doesn’t match the desired state, kube-controller-manager will attempt to make changes to achieve the desired state. 
It’s called the “controller manager” because many Kubernetes objects are maintained by loops of code called controllers. These loops of code handle the process of remediation.

Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.
Some types of these controllers are:
 - Node controller: Responsible for noticing and responding when nodes go down.
 - Job controller: Watches for Job objects that represent one-off tasks, then creates Pods to run those tasks to completion.
 - Endpoints controller: Populates the Endpoints object (that is, joins Services & Pods).
 - Service Account & Token controllers: Create default accounts and API access tokens for new namespaces.

#### cloud-controller-manager
kube-cloud-manager manages controllers that interact with underlying cloud providers.  
For example, if you manually launched a Kubernetes cluster on Google Compute Engine, kube-cloud-manager would be responsible for bringing in Google Cloud features like load balancers and storage volumes when you needed them.  
If you are running Kubernetes on your own premises, or in a learning environment inside your own PC, the cluster does not have a cloud controller manager.

### Node components
Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

#### Kubelet
An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.
The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. 

When the kube-apiserver wants to start a Pod on a node, it connects to that node’s kubelet.  
Kubelet uses the container runtime to start the Pod and monitors its lifecycle, including readiness and liveness probes, and reports back to Kube-APIserver.

The kubelet doesn't manage containers which were not created by Kubernetes.

#### kube-proxy 
kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.
kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.
kube-proxy uses the operating system packet filtering layer if there is one and it's available. Otherwise, kube-proxy forwards the traffic itself.

#### Container runtime
The container runtime is the software that is responsible for running containers.
Kubernetes supports several container runtimes: Docker, containerd, CRI-O, and any implementation of the Kubernetes CRI (Container Runtime Interface).
