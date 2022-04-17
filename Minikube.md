# Installation
https://minikube.sigs.k8s.io/docs/start/

# Working with minikube
**Start/stop your cluster**
```shell
minikube start
minikube stop
```

**Interact with your cluster**

minikube can download the appropriate version of kubectl and you should be able to use it like this:
```shell
minikube kubectl -- get po -A
```

If you already have kubectl installed, you can now use it to access your new cluster:
```shell
kubectl get po -A
```

on linux, you can create an alias to make life easier
```shell
alias kubectl="minikube kubectl --"
```

 minikube bundles the Kubernetes Dashboard, allowing you to get easily acclimated to your new environment:
 ```shell
minikube dashboard
```

> Note: Initially, some services such as the storage-provisioner, may not yet be in a Running state. This is a normal condition during cluster bring-up, and will resolve itself momentarily.

# Create a deployment
`minikube kubectl -- create -f deployment.yml --dry-run=client`
