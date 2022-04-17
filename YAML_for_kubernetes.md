# YAML for K8s
> __It covers only k8s dependent YAML specification.__

Ref: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#-strong-api-overview-strong-

## What is YAML
YAML is called as ain't markup language or Yet Another Markup Language, and is actually a data serialization language instead of a just markup language like html.

## Kubernetes YAML structure
```yml
apiVersion: 
kind: 
metadata: 
spec:
```
### apiVersion
Kubernetes have different versions of API for each objects. We discuss about API in detail in upcoming sessions. For now, lets keep it as simple as possible.

Pod is one of the kind of object which is part of core v1 API So for a Pod, we usually see `apiVersion: v1`

### kind
As explained above we specify the kind of API object with `kind:` field.

### metadata
We have seen the use of metadata earlier.
As the name implies , we usually store name of object and labels in `metadata` field.

### spec
Object specification will go hear. 
The specification will depend on the kind and apiVersion we use

**Lets explore Pod spec yaml** 
```yml
apiVersion: v1 
kind: Pod 
metadata: 
  name: coffee-app01 
  labels: 
    app: frontend 
    run: app01 
spec: 
  containers: 
  - name: demo 
    image: ngnix
```
In above specification , you can see that we have specified name and labels in matadata field.
The spec starts with cotainer field and we have added a container specification under it.

You might be wondering , how can we memories all these options. In reality , you don’t have to.

So lets discuss about command `kubectl explain` so that we don’t have to remember all YAML specs of kubernetes objects.

With `kubectl explain` subcommand , you can see the specification of each objects and can use that as a reference to write your YAML files.

**Fist level spec**
We will use `kubectl explain Pod` command to see the specifications of a Pod YAML.

`kubectl explain pod`
```
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec <Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

   status       <Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
```

As we discussed earlier , the specification is very familiar.
Field status is `readonly` and its system populated , so we don’t have to write anything for status.

**Exploring inner fields**
If we want to see the fields available in spec , then execute below command.  
`$kubectl explain pod.spec`
--- command o/p

As you can see in spec the containers filed is -required- which indicates that this filed is mandatory.
<[]Object> indicates that its an array of objects , which means , you can put more than one element under containers
That make sense , because the Pod may contain more than one container.
In YAML we can use - in front of a field to mark it as an array element.

If you want to add one more container in Pod , we will add one more array element with needed values.
```yml
apiVersion: v1 
kind: Pod 
metadata: 
  name: app01 
  labels: 
    app: frontend 
    run: app01 
spec: 
  containers: 
  - name: demo 
    image: ngnix 
  - name: demo2 
    image: httpd
```
How do I know the containers array element need name and image ?
We will use explain command to get those details.   

`$kubectl explain pod.spec.containers`
```
KIND:     Pod
VERSION:  v1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

     PodSpec is a description of a pod.

FIELDS:
   activeDeadlineSeconds        <integer>
     Optional duration in seconds the pod may be active on the node relative to
     StartTime before the system will actively try to mark it failed and kill
     associated containers. Value must be a positive integer.

   affinity     <Object>
     If specified, the pod's scheduling constraints

   automountServiceAccountToken <boolean>
     AutomountServiceAccountToken indicates whether a service account token
     should be automatically mounted.

   containers   <[]Object> -required-
     List of containers belonging to the pod. Containers cannot currently be
     added or removed. There must be at least one container in a Pod. Cannot be
     updated.

   dnsConfig    <Object>
     Specifies the DNS parameters o...
```

> As you can see , name and image are of type string which means , you have to provide a string value to it.
