# Pod
Pods are the basic unit for running containers inside kubernetes.  
A pod provides a way to set environment variables, mount storage and feed other information into a container.  

In Kubernetes, Pods are responsible for running your containers. 
Every Pod holds at least one container, and controls the execution of that container. When the containers exit,the Pod dies too.  
![image](https://user-images.githubusercontent.com/41310048/137701738-624dc8b4-4c52-46b8-90c4-1c7a44b5d595.png)

If there is more than one container in a pod, they are tightly coupled and share resources including networking and storage.  
Kubernetes assigns each Pod a unique IP address. Every container within a Pod shares the network namespace, including IP address and network ports.  
Containers within the same Pod can communicate through localhost, 127.0.0.1. A Pod can also specify a set of storage Volumes, to be shared among its containers.  
> Be careful: Pods are not self-healing.

# Node 
A worker machine in Kubernetes, part of a cluster.
# Cluster
A set of Nodes that run containerized applications managed by Kubernetes.
nodes in the cluster are not part of the public internet.
# Edge router
A router that enforces the firewall policy for your cluster. This could be a gateway managed by a cloud provider or a physical piece of hardware
# Cluster network
A set of links, logical or physical, that facilitate communication within a cluster according to the Kubernetes [networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/).
# Service 
A Kubernetes [Service](https://kubernetes.io/docs/concepts/services-networking/service/) that identifies a set of Pods using [label](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels) selectors. Unless mentioned otherwise, Services are assumed to have virtual IPs only routable within the cluster network.

# Secret
A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key.