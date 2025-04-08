K8s follows master and worker node architecture, master node manages the worker nodes and the workloads (Pods) in the cluster.
Master node (also referred as control plane) make global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events, while worker nodes host the Pods that are the components of the application workload.

![](Pasted%20image%2020250407193123.png)

![](Pasted%20image%2020250408172359.png)

Ref: [Cluster Architecture | Kubernetes](https://kubernetes.io/docs/concepts/architecture/)

## K8S Components
**Control plane, or Master Node components**

| Component name           | Description                                                                                                |
| :----------------------- | :--------------------------------------------------------------------------------------------------------- |
| kube-apiserver           | Component on the master that exposes the K8S API.                                                          |
| etcd                     | Database, that stores data in Key-vaule pairs for all cluster data.                                        |
| kube-scheduler           | component that watches newly created pods that have no node assigned and selects a node for them to run on |
| kube-controller-manager  | Responsible for controlling various aspects, including node controller, replication controller etc..       |
| cloud-controller-manager | Runs controllers that interact with the underlying cloud providers.                                        |

**Worker Node components**

| Component name    | Description                                                                                             |
| :---------------- | :------------------------------------------------------------------------------------------------------ |
| kubelet           | An agent that runs on each node in the cluster. it make sure the PODs are running in the node           |
| kube-proxy        | Acts as a network proxy which maintains network rules on the host and performing connection forwarding. |
| Container runtime | Software which is responsible for running containers, ex: docker, containerD.                           |

# K8S Components

