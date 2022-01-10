# K8s namespaces

list of namespaces
`kubectl get ns`

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
> it creates the context, if its doesnt exist

set the context
`kubectl config use-context my-context-1`

## Delete a name space
`kubectl delete ns <my-ns1>`
> Deleting a name space will delete all the resources in it.

## resource quotas