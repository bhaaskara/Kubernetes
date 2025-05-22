
## ClusterIP
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

![[Pastedimage20220107151525.png]](/Images/Pastedimage20220107151525.png)
-

## Node Port
- Exposes the Service on each Node's IP at a static port (the `NodePort`). 
- access the service/app from external world.
- You'll be able to contact the `NodePort` Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`.
- Node port range - 30000 - 32768

### Node port limitations
- need to track which nodes have pods with exposed ports.  
- it only exposes one service per port.
- the ports available to NodePort are in the 30,000 to 32,767 range

## Load balancer
This is the default method for many Kubernetes installations in the cloud, and it works great. It supports multiple protocols and multiple ports per service. But by default it uses an IP for every service, and that IP is configured to have its own load balancer configured in the cloud. These add costs and overhead that is overkill for essentially every cluster with multiple services, which is almost every cluster these days.

Exposes the Service externally using a cloud provider's load balancer. `NodePort` and `ClusterIP` Services, to which the external load balancer routes, are automatically created.

Load balancer module only works out-of-the-box on the largest public clouds for various reasons.

### Load Balancers in the Cloud vs on Bare Metal
There is an advantage to using the Kubernetes Load Balancer feature on the biggest public clouds. Through the cloud controller, Kubernetes will automatically provision and deprovision the required external IP and associated load balancer, and the nodes it will connect to in the cluster.

If you are running on-premises, especially on bare metal, there is no load-balancer service and pool of IPs just sitting idle and waiting for general usage. This is where the open source project [MetalLB](https://metallb.universe.tf/) comes into play. It has been designed from the ground up to specifically address this need for Kubernetes. It is still a fairly new project and requires experience to make it stable and reliable. Platform9 includes MetalLB and has the expertise to support on-premise deployments.

MetalLB even runs within Kubernetes to make it easier to manage and maintain. To use MetalLB you need a pool of IP addresses it can distribute, and a few ports open. It even supports working with Border Gateway Protocol ([BGP](https://www.cloudflare.com/en-ca/learning/security/glossary/what-is-bgp/)) for more complex networking scenarios. For example, when multiple Kubernetes clusters are involved in the environment. 

In these multi-cluster scenarios, MetalLB manages the pool of IPs across the clusters including which cluster is primary, secondary, and tertiary for specific services.

## ExternalName
Maps the Service to the contents of the `externalName` field (e.g. `foo.bar.example.com`), by returning a `CNAME` record with its value. No proxying of any kind is set up.

> You can also use [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) to expose your Service. Ingress is not a Service type, but it acts as the entry point for your cluster. It lets you consolidate your routing rules into a single resource as it can expose multiple services under the same IP address

## Ingress
While ingress – in normal networking functionality – refers to any inbound traffic, in Kubernetes it strictly refers to the API that manages traffic routing rules like SSL termination. The ingress controller in Kubernetes is the application that is deployed to implement those rules.

Ingress isn’t a service type like NodePort, ClusterIP, or LoadBalancer. Ingress actually acts as a proxy to bring traffic into the cluster, then uses internal service routing to get the traffic where it is going. Under the hood, Ingress will use a NodePort or LoadBalancer service to expose itself to the world so it can act as that proxy.

Here is an example of how ingress works: A deployment would define a new service. It would then tell Ingress that new.app.example.com is the external DNS that maps to the service. The service wants to receive traffic on TCP port 8181. The Ingress controller then sets up those rules so when it receives a request asking for new.app.example.com:8181, it knows where to send the payload and URI information for processing. 

The actual rules can get much more complicated. Out-of-the-box, they typically stick to layer 4 requests like the example above; although layer 7 requests involving cookie paths and specific query parameters on the URI are becoming more prevalent; especially when a service mesh is involved. Service meshes, like Istio, allow very fine-grained control of how traffic is sent to one or more versions of a service – including blue/green, AB, Canary, or even payload-based. 

As an additional benefit, service meshes can even route services between Kubernetes clusters without using Ingress or any of the other methods discussed here.

Commonly used ingress controllers are NGINX, Contour, and HAProxy.

## Which to use
|     | Node port | Load balancer | Ingress |
| :-- | :--  | :-- |:-- |
| Supported by core Kubernetes | Yes | Yes | Yes |
| Works on every platform Kubernetes will deploy | Yes | Only supports a few public clouds. MetalLB project allows use on-premises. | Yes |
| Direct access to service | Yes | Yes | No |
| Proxies each service through third party (NGINX, HAProxy, etc) | No | No | Yes |
| Multiple ports per service | No | Yes | Yes |
| Multiple services per IP | Yes | No | Yes |
| Allows use of standard service ports (80, 443, etc) | No | Yes | Yes |
| Have to track individual node IPs | Yes | No | Yes, when using NodePort; No, when using LoadBalancer |

NodePort wins on simplicity, but you need to open firewall rules to allow access to ports 30,000 to 32,767, and know the IPs of the individual worker nodes.

LoadBalancer when on a public cloud, or supported by MetalLB, works great with the service being able to control the exact port it wants to use. The downside is it can get expensive, as every service will get its own load balancer and external IP, which cost `$$$` on the public cloud.

Ingress is becoming the most commonly used, combined with the load balancer service; especially with MetalLB now available, as it minimizes the number of IPs being used while still allowing for every service to have its own name and/or URI routing.

# Service end points
K8s creates a end point object same name as service, to track of which pods are the members/end points of the service.
The end point object is dynamic and gets updated when pods get destroyed/created.

List the end points with below command.
`kubectl get endpoints`

Service node port Range - 30000-32768
> By default, the range of the service NodePorts is **30000-32767.   
    This range contains 2768 ports, which means that you can create up to 2768 services with  
    NodePorts.

### Over Capacity Endpoints[](https://kubernetes.io/docs/concepts/services-networking/service/#over-capacity-endpoints)
If an Endpoints resource has more than 1000 endpoints then a Kubernetes v1.22 (or later) cluster annotates that Endpoints with `endpoints.kubernetes.io/over-capacity: truncated`. This annotation indicates that the affected Endpoints object is over capacity and that the endpoints controller has truncated the number of endpoints to 1000

## Multi port service
![[multi_port_service.png]](/Images/multi_port_service.png)
```yml
apiVersion: v1
kind: service
metadata:
    name: mongodb-service
spec:
    selector:
        app: mongodb
    ports:
        - name: mongodb-server   #Mandatory in multiport config
          protocol: TCP
          port: 27017
          targetport: 27017
        - name: mongodb-exporter
          protocol: TCP
          port: 9216
          targetport: 9216 
```

- `-name` field is optional in single port service and mandatory in multi port service.

## Headless service
...

## Services without selectors[](https://kubernetes.io/docs/concepts/services-networking/service/#services-without-selectors)

Services most commonly abstract access to Kubernetes Pods, but they can also abstract other kinds of backends. For example:

-   You want to have an external database cluster in production, but in your test environment you use your own databases.
-   You want to point your Service to a Service in a different [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces) or on another cluster.
-   You are migrating a workload to Kubernetes. While evaluating the approach, you run only a portion of your backends in Kubernetes.

In any of these scenarios you can define a Service _without_ a Pod selector. For example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

Because this Service has no selector, the corresponding Endpoints object is not created automatically. You can manually map the Service to the network address and port where it's running, by adding an Endpoints object manually:

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```

# Ingress
![](Pasted%20image%2020220422195059.png)

Example Ingress YAML
![](Pasted%20image%2020220422195421.png)
Note: https is not yet configured.

![](Pasted%20image%2020220422195707.png)

- Here Internal service type is default - clusterIP.

![](Pasted%20image%2020220422200052.png)
![](Pasted%20image%2020220422200130.png)

## Ingress implementation
### Ingress controller
- evaluates all the rules
- manages redirections
- entry point to the cluster
- Many third party implementation are available and Nginix ingress controller is one of the popular ones.

`Need to implement ingress controller that evaluates and process ingress rules, for the ingress to work` 

![](Pasted%20image%2020220422200411.png)

![](Pasted%20image%2020220422200511.png)

## Ingress on cloud
![](Pasted%20image%2020220422201040.png)
## Ingress on bare metal
![](Pasted%20image%2020220422201153.png)
**Proxy server**
- separate server
- Assigned with public IP address and open ports
- entry point to the cluster

**_no server in k8s cluster is accessible externally_**

![](Pasted%20image%2020220422201610.png)

# QA
how to find the individual pods IPs through NS lookup of service
how the load balancer works, as it uses target port and node port and cluster IP.
how to configure ingress
how to configure load balancer(i.e cloud)

# References
https://www.youtube.com/watch?v=T4Z7visMM4E&list=PLVjlqAPNDEDZ9_8md_qi-ZHaQIj9mVCVw&index=3

