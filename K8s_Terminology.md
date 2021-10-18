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
