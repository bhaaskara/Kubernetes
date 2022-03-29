# What is a k8s service
In Kubernetes, a Service is an abstraction which defines a logical set of Pods and a policy by which to access them. 
The set of Pods targeted by a Service is usually determined by a [selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/).

`k8s def:` An abstract way to expose an application running on a set of [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) as a network service.

# Different services types
- Cluster IP - Default type
- Node port
- Load balancer
- Headless

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
- pods are identified by the selectors, i.e pod labels  
- all the selectors should match with the pod labels, not just one of them

## Node Port
- Exposes the Service on each Node's IP at a static port (the `NodePort`). 
- access the service/app from external world.
- You'll be able to contact the `NodePort` Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`.
- Node port range - 30000 - 32768


## Load balancer
Exposes the Service externally using a cloud provider's load balancer. `NodePort` and `ClusterIP` Services, to which the external load balancer routes, are automatically created.

## ExternalName
Maps the Service to the contents of the `externalName` field (e.g. `foo.bar.example.com`), by returning a `CNAME` record with its value. No proxying of any kind is set up.

> You can also use [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) to expose your Service. Ingress is not a Service type, but it acts as the entry point for your cluster. It lets you consolidate your routing rules into a single resource as it can expose multiple services under the same IP address


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



## QA
how to find the individual pods IPs through NS lookup of service
how the load balancer works, as it uses target port and node port and cluster IP.
how to configure ingress
how to configure load balancer(i.e cloud)

# References
https://www.youtube.com/watch?v=T4Z7visMM4E&list=PLVjlqAPNDEDZ9_8md_qi-ZHaQIj9mVCVw&index=3

