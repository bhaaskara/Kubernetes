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

## Restart a POD
First, let’s talk about some reasons we might restart pods:
- **Resource use isn’t stated or when the software behaves in an unforeseen way.** If a container with 600 Mi of memory attempts to allocate additional memory, the pod will be [terminated with an OOM](https://www.containiq.com/post/oomkilled-troubleshooting-kubernetes-memory-requests-and-limits). You must restart your pod in this situation after [modifying resource specifications](https://www.containiq.com/post/setting-and-rightsizing-kubernetes-resource-limits).
- **A pod is stuck in a terminating state.** This is found by looking for pods that have had all of their containers terminated yet the pod is still functioning. This usually happens when a cluster node is taken out of service unexpectedly, and the cluster scheduler and controller-manager cannot clean up all the pods on that node.
- **An error can’t be fixed.**
- **Timeouts.**
- **Mistaken deployments.**
- **Requesting** [**persistent volumes**](https://www.containiq.com/post/kubernetes-persistent-volumes) **that are not available.**

> You can use docker restart {container_id} to restart a container in the Docker process, but there is no restart command in Kubernetes. In other words, there is no kubectl restart {podname}.

**Method 1:** kubectl replace
`kubectl get pod <pod_name> -n <namespace> -o yaml | kubectl replace --force -f -`

If there is no YAML file and the pod object is started, it cannot be directly deleted or scaled to zero, but it can be restarted by the above command. 
The meaning of this command is to get the YAML statement of currently running pods and pipe the output to kubectl replace, the standard input command to achieve the purpose of a restart.

**Method 2:** kubectl delete pod
If the pod is part of a deployment, upon deleting, the pod will be recreated by deployment.
`kubectl delete pod <pod_name> -n <namespace>`

**Method 3:** kubectl rollout restart
The [controller](https://www.containiq.com/post/kubernetes-controllers/) kills one pod at a time, relying on the ReplicaSet to scale up new pods until all of them are newer than the moment the controller resumed. Rolling out restart is the ideal approach to restarting your pods because your application will not be affected or go down.
`kubectl rollout restart deployment <deployment_name> -n <namespace>`

**Method 4:** Kubectl scale
If there is no YAML file, a quick solution is to scale the number of replicas using the kubectl command scale and set the replicas flag to zero
`kubectl scale deployment <deployment_name> --replicas=0 -n <ns_name>`

To restart the pod, set the number of replicas to at least one
`kubectl scale deployment <deployment_name> --replicas=1 -n <ns_name>`

## Schedule PODs on master node
1. Remove the taint on master node.
    `kubectl taint nodes --all node-role.kubernetes.io/master-`
2. Test the pod Schedule on master
    1. Create a nginx deployment with replicas equal to number of nodes.
```yml
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: nginx 
  labels: 
    app: nginx 
    color: green 
spec: 
  replicas: 2 
  selector: 
    matchLabels: 
      app: nginx
  template: 
    metadata: 
      labels: 
        app: nginx 
        color: green 
    spec: 
      containers: 
      - name: nginx 
        image: nginx:latest
```           
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

