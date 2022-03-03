# Cluster
## Reset master/worker
> when cluster is not in a desired state or unable to access, may need to reset  

`kubeadm reset`
# Pods
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
## Create a static Pod
Create the staticpod definition yml file under `/etc/kubernetes/manifests` on node, where you want to create the static pod. 
```yml
apiVersion: V1
kind: pod
metadata:
  name: staticpod
  labels:
    type: static
spec:
  containers:
    - name: staticpodcontainer
      image: httpd
```
> Static pods contains its node name postfixes in pod's name.

`kubectl get pods -A`  
>NAMESPACE     NAME                             READY   STATUS    RESTARTS        AGE  
>**default       staticpod-worker2**                1/1     Running   0               2m12s  
>kube-system   coredns-78fcd69978-lnprp         1/1     Running   1 (3h35m ago)   49d  
>kube-system   coredns-78fcd69978-v9tt9         1/1     Running   1 (3h35m ago)   49d  
>**kube-system   etcd-master**                      1/1     Running   0               125m  

>you can also use `docker ps` on a node to check the pod.  
>Remove the yml file to delete static pod.

# K8s nodes upgrade
- check the release notes of new version
- Upgrading from 1.16 to 1.18
- Check for API version compatibility 

## Upgrade path
- upgrade the master first (1.16 -> 1.17 -> 1.18) it would take an hour
    - on cloud provider console just initiate the master upgrade.
- Upgrade the nodes (1.16 -> 1.18)
    - Verify the existing nodes
    - `kubectl get nodes`
    - Drain the node
    - `kubectl drain <node name> --ignore-daemonsets --delete-local-data `
    - __Drain a node will cordon the node aswell__
    -                                                                          

# K8s Issues and trouble shooting
## Pod trouble shooting
```
kubectl logs <pod>
kubectl events
kubectl exec -it
kubectl describe <pod>
```

## Memory issue
check for the resource usage in k8s dashboard or `kubectl events`
if you want to increase the memory then edit the resources under deployment and apply.

