# What is a k8s service
An abstract way to expose an application running on a set of [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) as a network service.

In Kubernetes, a Service is an abstraction which defines a logical set of Pods and a policy by which to access them. 
The set of Pods targeted by a Service is usually determined by a [selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/).

# Different services types
- Cluster IP service
- Node port service
- Load balancer service
- Headless service

## ClusterIP service
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

## Service end points
K8s creates a end point object same name as service, to track of which pods are the members/end points of the service.
The end point object is dynamic and gets updated when pods get destroyed/created.

List the end points with below command.
`kubectl get endpoints`

Service node port Range - 30000-32768
> By default, the range of the service NodePorts is **30000-32767.   
    This range contains 2768 ports, which means that you can create up to 2768 services with  
    NodePorts.

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




## QA
how to find the individual pods IPs through NS lookup of service
how the load balancer works, as it uses target port and node port and cluster IP.
how to configure ingress
how to configure load balancer(i.e cloud)

# References
https://www.youtube.com/watch?v=T4Z7visMM4E&list=PLVjlqAPNDEDZ9_8md_qi-ZHaQIj9mVCVw&index=3
