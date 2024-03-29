# Cluster
## Reset master/worker
> when cluster is not in a desired state or unable to access, may need to reset  

`kubeadm reset`

# Pods
## Connect to a Running Container
```sh
kubectl exec -it <pod_name> -- /bin/bash 
```
Running individual commands in a container
```sh
kubectl exec <pod_name> env
```
examples
```sh
kubectl exec shell-demo -- ps aux
kubectl exec shell-demo -- ls /
kubectl exec shell-demo -- cat /proc/1/mounts
```
## connect to a container when a pod has more than one container
```sh
kubectl exec -i -t <my-pod> --container <main-app> -- /bin/bash
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

## HPA
https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
```
1. Install Metrics Server
2. Deploy Deployment, Service, HPA
3. Increase Load
4. check Pod scale!

Note - Need node larger than t3.micro, recommended m5.large on AWS
```

**1. Install metrics server**
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
**2. Deploy Deployment, service, HPA**
    
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
                                                                           

# Service
## Ingress
### Configure ingress in minikube
![](Pasted%20image%2020220423154808.png)

```
1. Install ngnix ingress controller
2. Create ingress rule
```

**1. Install Nginx ingress controller**
`minikube addons enable ingress`
Automatically starts the k8s nginx implementation of ingress controller

check the status
`kubectl get pods -n kube-system`

**2. Create ingress rule**
Create an ingress rule to access the minikube dashboard externally

Get the kubernetes dashboard details
`kubectl get all -n kubernetes-dashboard`

```yml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kubernetes-dashboard
spec:
  rules:
  - host: dashboard.com
    http:
      paths:
      - backend:
          serviceName: kubernetes-dashboard #kubernetes internal service name
          servicePort: 80
```
`kubectl apply -f ingress.yml`
`kubectl get ingress -n kubernetes-dashboard`
Note: wait for some time for the ip to be assigned

add the ingress ip in hosts file for it to resolve
```sh
vi /etc/hosts

...
192.168.64.5    dashboard.com
```

### Configure default backend
![](Pasted%20image%2020220423161123.png)
this is in general used to configure some custom error message.

### multiple paths for same host
![](Pasted%20image%2020220423161544.png)

### Multiple sub-domains or domains
![](Pasted%20image%2020220423161754.png)

### Configuring tls certificate
![](Pasted%20image%2020220423161915.png)
![](Pasted%20image%2020220423162017.png)

# K8s Storage
## Create a custom storage class
```
apiVersion: storage.k8s.io/v1

kind: StorageClass

metadata:

name: managed-premium-retain-sc

provisioner: kubernetes.io/azure-disk

reclaimPolicy: Retain # Default is Delete, recommended is retain

volumeBindingMode: WaitForFirstConsumer # Default is Immediate, recommended is WaitForFirstConsumer

allowVolumeExpansion: true

parameters:

storageaccounttype: Premium_LRS # or we can use Standard_LRS

kind: managed # Default is shared (Other two are managed and dedicated)

##############################################################################

# Note-1:

#volumeBindingMode: Immediate - This setting implies that the PersistentVolumecreation,

#followed with the storage medium (Azure Disk in this case) provisioning is triggered as

#soon as the PersistentVolumeClaim is created.

# Note-2:

# volumeBindingMode: WaitForFirstConsumer

#By default, the Immediate mode indicates that volume binding and dynamic provisioning

#occurs once the PersistentVolumeClaim is created. For storage backends that are

#topology-constrained and not globally accessible from all Nodes in the cluster,

#PersistentVolumes will be bound or provisioned without knowledge of the Pod's scheduling

#requirements. This may result in unschedulable Pods.

#A cluster administrator can address this issue by specifying the WaitForFirstConsumer

#mode which will delay the binding and provisioning of a PersistentVolume until a

#Pod using the PersistentVolumeClaim is created. PersistentVolumes will be selected or

#provisioned conforming to the topology that is specified by the Pod's scheduling

#constraints.

##############################################################################

# Note-3:

#reclaimPolicy: Delete - With this setting, as soon as a PersistentVolumeClaim is deleted,

#it also triggers the removal of the corresponding PersistentVolume along with the

#Azure Disk.

#We will be surprised provided if we intended to retain that data as backup.

# reclaimPolicy: retain - Disk is retained even when PVC is deleted - Recommended Option

# Note-4:

# Both reclaimPolicy: Delete and volumeBindingMode: Immediate are default settings

##############################################################################

# Note-5:

# Additional Reference

# https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-disk

# Managed: When managed used, that disk is persisted for the Lifecycle of the cluster.

# If we delete cluster, it will delete the disk

##############################################################################
```

## Create a PVC
```sh
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium-retain-sc
  resources:
    requests:
    storage: 5Gi
```

# K8s Backup and Restore
## etcd
- All Kubernetes objects are stored on etcd 
- Periodically backing up the etcd cluster data is important to recover Kubernetes clusters under 
disaster scenarios 
- The snapshot file contains all the Kubernetes states and critical information 
- etcd supports built-in snapshot, so backing up an etcd cluster is easy 
- Restore from the snapshot
	`ETCDCTL_API=3 etcdctl snapshot restore snapshotdb`
	ETCD api version can be found with - `etcdctl version`

## Cluster certificates
- First need to have CA Authority to create Certs 
- Client certificates 
	- Admin Client certificate 
	- Kubelet Client certificate 
	- Controller Manager Client 
	- Kube Proxy Client 
	- Kube Scheduler Client 
- API Server Certificates 
- Service Account Key pairs 
- Move the files onto the appropriate servers 

# K8s Upgrade
##  Upgrade Control plane nodes
- Determine which version to upgrade to 
- Upgrading control plane nodes 
	- On your first control plane node, upgrade kubeadm: 
	- Drain the control plane node: 
	- On the control plane node, run: sudo kubeadm upgrade plan 
	- Choose a version to upgrade to and run : `sudo kubeadm upgrade apply v1.18.x` 
	- `kubectl uncordon <cp-node-name>` 
- Upgrade additional control plane nodes 
	- sudo kubeadm upgrade node 
##  Upgrade worker node kubeadm
- Upgrade kubeadm on all worker nodes 
- Drain the node: `kubectl drain <node-to-drain> —ignore-daemonsets` 
- Execute the upgrade cmd: `sudo kubeadm upgrade node` 
- Upgrade kubelet and kubectl 
- Restart the kubelet 
- Uncordon the node 
- Verify the status of the cluster 
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

