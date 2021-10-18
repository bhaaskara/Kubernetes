# Cluster
## Reset master/worker
> when cluster is not in a desired state or unable to access, may need to reset  

`kubeadm reset`
# Pods
## Create a static Pod
```sh

## Connect to a Running Container
```sh
kubectl exec -it shell-demo -- /bin/bash 
```
Running individual commands in a container
```sh
kubectl exec shell-demo env
```
examples
```sh
kubectl exec shell-demo -- ps aux
kubectl exec shell-demo -- ls /
kubectl exec shell-demo -- cat /proc/1/mounts
```
## connect to a container when a pod has more than one container
```sh
kubectl exec -i -t my-pod --container main-app -- /bin/bash
```
