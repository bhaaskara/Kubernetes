# YAML for K8s
> __It covers only k8s dependent YAML specification.__

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

So lets discuss about command kubectl explain so that we don’t have to remember all YAML specs of kubernetes objects.

With kubectl explain subcommand , you can see the specification of each objects and can use that as a reference to write your YAML files.

**Fist level spec**
We will use `kubectl explain Pod` command to see the specifications of a Pod YAML.

-- command o/p

As we discussed earlier , the specification is very familiar.
Filed status is `readonly` and its system populated , so we don’t have to write anything for status.

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
--- cmd o/p
> As you can see , name and image are of type string which means , you have to provide a string value to it.
