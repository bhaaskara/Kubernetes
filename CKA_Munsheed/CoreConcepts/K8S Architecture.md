K8s follows master and worker node architecture, master node manages the worker nodes and the workloads (Pods) in the cluster.
Master node (also referred as control plane) make global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events, while worker nodes host the Pods that are the components of the application workload.

![](Pasted%20image%2020250407193123.png)

![](Pasted%20image%2020250408172359.png)

Ref: [Cluster Architecture | Kubernetes](https://kubernetes.io/docs/concepts/architecture/)

# K8S Components

## Control plane, or Master Node components
Summary

| Component name           | Description                                                                                                |
| :----------------------- | :--------------------------------------------------------------------------------------------------------- |
| kube-apiserver           | Component on the master that exposes the K8S API.                                                          |
| etcd                     | Database, that stores data in Key-vaule pairs for all cluster data.                                        |
| kube-scheduler           | component that watches newly created pods that have no node assigned and selects a node for them to run on |
| kube-controller-manager  | Responsible for controlling various aspects, including node controller, replication controller etc..       |
| cloud-controller-manager | Runs controllers that interact with the underlying cloud providers.                                        |
|                          |                                                                                                            |
Control plane components can be run on any machine in the cluster. However, for simplicity, set up scripts typically start all control plane components on the same machine, and do not run user containers on this machine.
Control plane setup that runs across multiple VMs.
[K8s HA](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/) 
### Kube-apiserver
- The API server is the front end for the Kubernetes control plane.  
- The core of Kubernetes' control plane is the API server and the HTTP API that it exposes.  
- Users, the different parts of your cluster, and external components all communicate with one another through the API server.  
- kube-apiserver also authenticates incoming requests, determines whether they are authorized and valid, and manages admission control.  
- The API Server is the only Kubernetes component that connects to etcd; all the other components
   must go through the API Server to work with the cluster state.
- Kubectl command’s job is to connect to kube-apiserver and communicate with it using the Kubernetes API. 
- But it’s not just kubectl that talks with kube-apiserver. In fact, any query or change to the cluster’s state must be addressed to the kube-apiserver.
- kube-apiserver is designed to scale horizontally, it scales by deploying more instances.  
  You can run several instances of kube-apiserver and balance traffic between those instances.
 
![](Pasted%20image%2020250409195138.png)

If you deploy your cluster using kubeadm (kubeadmin) tool it deploys kube=api server as POD in the kube-system name space on the master node.
View the kube-api server using `kubectl get pods -n kube-system` command.
The POD definition file for kube-api server is at: /etc/kubernetes/manifests/kube-apiserver.yaml

### etcd
etcd is a distributed reliable key-value store.

In a Linux environment, all the configurations are stored in the /etc directory.
etcd is inspired from that and there is an addition of d which is for distributed.

etcd reliably stores the configuration data of the Kubernetes cluster, representing the state of the
cluster (what nodes exist in the cluster, what pods should be running, which nodes they are
running on, and a whole lot more) at any given point of time.

- A Kubernetes cluster stores all its data in etcd.
- Any node crashing or process dying causes values in etcd to be changed.
- Whenever you create something with kubectl create / kubectl run will create an entry in the etcd.

**Note:** You never interact directly with etcd; instead, kube-apiserver interacts with the database on behalf of the rest of the system.
etcdctl is the command line tool to interact with etcd, but recommended not to use it.
The API Server is the only Kubernetes component that connects to etcd.
### Kube-control manager
Control Plane component that runs controller processes.
kube-controller-manager has a broader job. It continuously monitors the state of a cluster through Kube-APIserver. Whenever the current state of the cluster doesn’t match the desired state, kube-controller-manager will attempt to make changes to achieve the desired state. 
It’s called the “controller manager” because many Kubernetes objects are maintained by loops of code called controllers. These loops of code handle the process of remediation.

Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.
Some of these controllers are:
 - Node controller: Responsible for noticing and responding when nodes go down.
 - Job controller: Watches for Job objects that represent one-off tasks, then creates Pods to run those tasks to completion.
 - Endpoints controller: Populates the Endpoints object (that is, joins Services & Pods).
 - Service Account & Token controllers: Create default accounts and API access tokens for new namespaces.
### Kube-scheduler
Control plane component that watches for newly created Pods with no assigned node, and selects a node for them to run on.  
But it doesn’t do the work of actually launching Pods on Nodes. Instead, whenever it discovers a Pod object that doesn’t yet have an assignment to a node, it chooses a node and simply writes the name of that node into the Pod object. 
Factors taken into account for scheduling decisions include:  
 - individual and collective resource requirements,  
 - hardware/software/policy constraints  
 - affinity (which cause groups of pods to prefer running on the same node)  
 - anti-affinity (which ensure that pods do not run on the same node) specifications  
 - data locality
 - inter-workload interference  
 - deadlines
### Cloud control manager
The cloud-controller-manager is a Kubernetes [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) component that embeds cloud-specific control logic. The cloud controller manager lets you link your cluster into your cloud provider's API, and separates out the components that interact with that cloud platform from components that only interact with your cluster.

By decoupling the interoperability logic between Kubernetes and the underlying cloud infrastructure, the cloud-controller-manager component enables cloud providers to release features at a different pace compared to the main Kubernetes project.

For example, if you manually launched a Kubernetes cluster on Google Compute Engine, kube-cloud-manager would be responsible for bringing in Google Cloud features like load balancers and storage volumes when you needed them.  
If you are running Kubernetes on your own premises, or in a learning environment inside your own PC, the cluster does not have a cloud controller manager.
## Worker Node components
Worker Node or simply Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

| Component name    | Description                                                                                             |
| :---------------- | :------------------------------------------------------------------------------------------------------ |
| kubelet           | An agent that runs on each node in the cluster. it make sure the PODs are running in the node           |
| kube-proxy        | Acts as a network proxy which maintains network rules on the host and performing connection forwarding. |
| Container runtime | Software which is responsible for running containers, ex: docker, containerD.                           |
### Kubelet
An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.
The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. 

When the kube-apiserver wants to start a Pod on a node, it connects to that node’s kubelet.  
Kubelet uses the container runtime to start the Pod and monitors its lifecycle, including readiness and liveness probes, and reports back to Kube-APIserver.

**The kubelet doesn't manage containers which were not created by Kubernetes.**
### kube-proxy (Optional)
kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.
kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.
kube-proxy uses the operating system packet filtering layer if there is one and it's available. Otherwise, kube-proxy forwards the traffic itself.

If you use a [network plugin](https://kubernetes.io/docs/concepts/architecture/#network-plugins) that implements packet forwarding for Services by itself, and providing equivalent behavior to kube-proxy, then you do not need to run kube-proxy on the nodes in your cluster.
### Container runtime
The container runtime is the software that is responsible for running containers.
Kubernetes supports several container runtimes: Docker, containerd, CRI-O, and any implementation of the Kubernetes CRI (Container Runtime Interface).
