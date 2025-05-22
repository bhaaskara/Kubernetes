# What is a k8s service
In Kubernetes, a Service is an abstraction which defines a logical set of Pods and a policy by which to access them. 

A “service” is defined as the combination of a group of pods, and a policy to access them
The set of Pods targeted by a Service is usually determined by a [selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/).

`k8s def:` An abstract way to expose an application running on a set of [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) as a network service.

# Different services types
- Cluster IP - Default type
- Node port
- Load balancer
- Headless

## Cluster IP
- Exposes the Service on a cluster-internal IP. 
- Choosing this value makes the Service only reachable from within the cluster.
- Default type
- type: no need to specify
```yml
apVersion: v1
kind: service
metadata:
  name: demo-service1
spec:                #no type specified here
  selector:          #pod selector lables
    app: service-one 
  ports:
   - protocol: tcp
     port: 3200       #on which we can access the service
     targetport: 3000 #Pod listening port
```
 - pods are identified by the selectors, i.e pod labels
 - **all the selectors should match with the pod labels, not just one of them**

## Node Port
- Exposes the Service on each Node's IP at a static port (the `NodePort`). 
- access the service/app from external world.
- You'll be able to contact the `NodePort` Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`.
- Node port range - 30000 - 32768

### Node port limitations
- need to track which nodes have pods with exposed ports.  
- it only exposes one service per port.
- the ports available to NodePort are in the 30,000 to 32,767 range

![](Pasted%20image%2020250512174751.png)
Note: Once you create the service it automatically selects the same node port across the nodes.

