





# POD
A Pod in Kubernetes represents a group of one or more application containers , and some shared
resources for those containers.
![](pod.png)

Containers within a Pod share an IP address and port space, and can find each other via localhost.

![](pod1.png)A - 

- Pod always runs on a Node.   
- A Node is a worker machine in Kubernetes.   
- Each Node is managed by the Master.   
- A Node can have multiple pods.


