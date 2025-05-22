# K8s namespaces
Namespaces are a way to organize clusters into virtual sub-clusters — they can be helpful when different teams or projects share a Kubernetes cluster. Any number of namespaces are supported within a cluster, each logically separated from others but with the ability to communicate with each other.

Namespaces provide a scope for names. Names of resources need to be unique within a namespace, but not across namespaces. Namespaces cannot be nested inside one another and each Kubernetes resource can only be in one namespace.

Namespaces are a way to divide cluster resources between multiple users (via [resource quota](https://kubernetes.io/docs/concepts/policy/resource-quotas/)).

It is not necessary to use multiple namespaces to separate slightly different resources, such as different versions of the same software: use [labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels) to distinguish resources within the same namespace.

## Default name spaces
K8s cluster comes with 4 default name spaces.
- default
- kube-system
- kube-public
- kube-node-lease

**kube-system**
- Do not create or modify in kube-system
- system process
- master and kubectl processes

**kube-public**
- contains publicly accessible data
- A configmap which contains cluster info
`kubectl cluster-info`

**Kube-node-lease**
- Is used by k8s for node heart beat.
- Each node has associated lease object in namespace
- Determines the availability of a node.

> kubernetes-dashboard name space comes bydefault with minikube installation, not in standard installation.

## Why to Use name spaces
- resource grouped in to name spaces.
- many teams with same application with different configuration.
- resource sharing, staging and development
   ![](Pasted%20image%2020220425171103.png)
   - resource sharing - Blue/green deployments
     ![](Pasted%20image%2020220425171230.png)
 - access and resource quotas (RAM,CPU and Storage)
 **_name spaces are not recommended for smaller projects with upto 10 users_**

## Characteristics of name spaces

Most of the resources can't be accesses across the name spaces
Each name space must define its own config map or secrets
Configmap that refers to a service can be shared among name spaces

Access service in another name space
![](Pasted%20image%2020220425172454.png)
Here the configmap will refer to a url post fixed with name space.
this is how we can access nginx or elastic stack service from another name space


## Working with Namespaces
list of namespaces
`kubectl get ns` or
`kubectl get namespace`

list pods across all name spaces
`kubectl get pods --all-namespaces`

Create a namespace from yaml
```
apiVersion: v1
kind: namespace
metadata:
    name: my-namespace-1
```

List the current config
`kubectl config view`

list the context
`kubectl config current-context`

create/set a context
`kubectl config set-context my-context-1 --namespace=my-ns1 --cluster=mycluser1 --user=admin`
> it creates the context, if it's doesn't exist

set the context
`kubectl config use-context my-context-1`

## Create resources in name space
`kubectl apply -f <file.yaml> --namespace=<ns-name>` or `kubectl create -f <yaml> -n <ns-name>`

Mention it in YAML
![](Pasted%20image%2020220425183215.png)

## Change active name space
change the active namespace or context using `kubens` 
this is a 3rd party tool `kubectx` and need to install.

once installed use `kubens` to list the name spaces and use `kubens <ns_name>` to set.

kubectx installation: [GitHub - ahmetb/kubectx: Faster way to switch between clusters and namespaces in kubectl](https://github.com/ahmetb/kubectx#installation)

### Delete a name space
`kubectl delete ns <my-ns1>`
> Deleting a name space will delete all the resources in it.

List the global resources / Not part of a name spce
`kubectl api-resources --namespace=false`

List the namespaced resources
`kubectl api-resources --namespace=true`

## Resource quotas
https://kubernetes.io/docs/concepts/policy/resource-quotas/

## Limit Range
Is used to limit the resource usage at pod/container level.