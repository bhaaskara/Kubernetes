# Kubernetes Storage
https://kubernetes.io/docs/concepts/storage/volumes/

Containers use the principle of immutability. This means that when containers are destroyed, all data created during the lifetime of the containers will also be destroyed. Immutability is not always appropriate, however. 
For example, applications that need to share information or maintain state cannot lose this information. In these cases, containers must have a location to permanently store information, which should outlive the lifespan of an individual container.

In Kubernetes, the most basic type of storage is non-persistent - which is known as ephemeral. Each container has ephemeral storage by default—this storage uses a temporary directory on the host machine that hosts the Kubernetes pod. It is portable, but not durable.

Kubernetes supports multiple types of persistent storage. This can include file, block, or object storage services from cloud providers (such as Amazon S3), storage devices in the local data center, or data services like databases. 
Kubernetes provides mechanisms to abstract this storage for applications running on containers, so that applications never communicate directly with storage media.

**Persistent Volume**: A piece of storage can be used for Kubernetes 
**Persistent Volume Claim**: A request for PV 
**Storage Class**: A collection of PV 
**Provisioner**: Used to provision PV 
**Volume**: Referring to the storage used by the Pod 

## Persistent volumes and   Persistent Volume Claims (PVC)
To enable persistent storage, Kubernetes uses two key concepts:

**PersistentVolume (PV)** is a storage element in a cluster, defined manually by an administrator or dynamically defined by a storage class. A PV has its own lifecycle, separate from the lifecycle of Kubernetes pods. The PV API captures storage implementation details for NFS, cloud provider-specific storage systems, or iSCSI.

**PersistentVolumeClaim (PVC)** is a user’s storage request. An application running on a container can request a certain type of storage. For example, a container can specify the size of storage it needs or the way it needs to access the data (read only, write, read/write, with one-time or ongoing access).

Beyond storage size and access mode, administrators can offer PVs with custom properties, such as the type of disk (HDD vs SSD), the level of performance, or the storage tier (regular or cold storage). Users can request storage based on these custom parameters without knowing the implementation details of the underlying storage. This is achieved using the StorageClass resource.

Storage activity | K8s storage primitive
:---------------  | :-------------------
Provisioning | Persistent volume
Configuration | Storage class
Attaching | Persistent volume claim

### Features of a persistent volume
- Provisioned dynamically or by an admin
- Has a particular size
- Created with a specific file system
- Has identity characteristics such as volume Id and a name.

 ### Pod and persistent volume
 ![[pod_volume.png]](/Images/pod_volume.png)
 
### Types of volumes
1. Ephemeral
2. Persistent
    - Awselasticblock 
    - Azuredisk 
    - Azurefile 
    - Cephfs
    - Cinder (is a volume provided by open stack)
    - Configmap (is a way to inject config data into pods)

### Volume binding mode
 - Immediate
 - Waitforfirstconsumer

### Volume and Claim lifecycle
 - Provisioning
 - Binding
 - using
 - Reclaiming

### PV access modes
- Read-Write-Once (RWO)
- Read-Write-Many (RWX)
- Read-Only-Many (ROX)

**Read-Write-Once** type storage only can be read/write on one node at any given time.
- High performance block device
- AWS EBS, Azure Disk, Google Persistent Disk, Ceph RBD, Longhorn

**Read-Write-Many** type storage can be read/write on multiple nodes at the same time.
- Distributed Filesystem
- AWS EFS, NFS, GlusterFS, CephFS

### Deployment vs StatefulSet   
- **Pods in one Deployment** share the same volume   
    - No matter which node the pod runs on   
    - Better suit for RWX type storage   
• **Each Pod in one StatefulSet** can have one volume   
    • VolumeClaimTemplate   
    • Better suit for RWO type storage

## Storage class
`StorageClass refers to a volume plugin, also called a provisioner`

You can configure StorageClass and assign PVs to each one. A StorageClass represents one type of storage. For example, one StorageClass may represent fast SSD storage, while another can represent magnetic drives, or remote cloud storage. This allows Kubernetes clusters to configure various types of storage according to workload requirements.

A StorageClass is a Kubernetes API for setting storage parameters using dynamic configuration to enable administrators to create new volumes as needed. The StorageClass defines the volume plug-in, external provider (if applicable), and the name of the container storage interface (CSI) driver that will enable containers to interact with the storage device. 

### Dynamic Provisioning of StorageClasses
Kubernetes supports dynamic volume configuration, allowing you to create storage volumes on demand. Therefore, administrators do not have to manually create new storage volumes and then create a PersistentVolume object for use in the cluster. When the user requests a specific type of storage, the entire process runs automatically.

The cluster administrator defines storage class objects as needed. Each StorageClass refers to a volume plugin, also called a provisioner. When a user creates a PVC, the provisioner automatically provisions a volume according to the required storage criteria.

```
apiVersion: storage.k8s.io/v1 
kind: StorageClass 
metadata : 
  name : storagel 
provisioner: kubernetes.io/aws—ebs 
parameters: 
  type: gp2 
reclaimPolicy: Retain 
allowVolumeExpansion: true 
mountOptions: 
  - debug 
volumeBindingMode: Immediate 
```

### Storage class parameters
 - Maximum limit of parameters for a StorageClass is 512 
 - Length must not exceed 256 KiB 
 - Key parameters 
    - Type 
    -  Zone 
    - iopsPerGB 
    - fsType 
    - Encrypted 
    - Kmskeyid

**Persistent storage in K8s before storage class**
![](Pasted%20image%2020220707233833.png)

**Persistent storage in K8s after storage class**
![](Pasted%20image%2020220707233904.png)



## Empty dir
> emptyDir is only for the current pod and can't be shared, and it will be deleted once the pod deleted.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-test-pd
spec:
  containers:
  - image: nginx
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

Create the pod
`kubectl apply -f <yml>`
`kubectl exec emptydir-test-pd df`
`kubectl exec -it emptydir-test-pd -- /bin/bash`
`kubectl describe pod emptydir-test-pd`

To know the path of the volume on the host machine
`docker inspect 3927c864300c03bb188ecb54991f3437796cf4afc8d0816f381f181e044f9a6d`
> it would be in general: /var/lib/kubelet/pods/156f280e-b951-4424-8516-d5c0539e3100/volumes/kubernetes.io~empty-dir/cache-volume

## Volumes
Volumes are basic entities in Kubernetes, used to provide storage to containers. A volume can support all types of storage, including network file system (NFS), local storage devices, and cloud-based storage services. You can also build your own storage plugins to support new storage systems. Access to volumes can be achieved directly via pods or through persistent volumes

```
The kubernetes volume in its simplest form is a directory that's attached to a pod and mounted 
to one or more containers within the pod.
```

## Container Storage Interface (CSI)
CSI is a Kubernetes extension that simplifies storage management. Before CSI, users needed to integrate the data store’s device driver with Kubernetes, which was quite complex. CSI provides an extensible plugin architecture, so you can easily add plugins that support the storage devices and services used in your organization.

## Host path

## Snapshots
### Role of volume snapshot
 - Represents a point-in-time copy of a volume 
 - Used to restore existing or provision new volumes 
 - Supported by multiple storage systems 

### Prereqs to use k8s snapshots
 - Deployed and running CSI driver implementing snapshots 
 - Enabled Kubernetes volume snapshotting feature 

### Steps to clone CSI volume
 - Implement checks for CSI plugin 
 - Implement CLONE VOLUME controller capability 
 - Deploy volume clone functionality 
 - Enable cloning for CSI volumes 

## Kubernetes Storage Best Practices
### Kubernetes Volumes Settings
The Persistent Volume (PV) life cycle is independent of any particular container in the cluster.  
Persistent Volume Claims (PVC) are a request made by a container user or application for a specific type of storage.  
When creating a PV, Kubernetes documentation recommends the following:
 - Always include PVCs in the container configuration.
 - Never include PVs in container configuration—because this will tightly couple a container to a specific volume.
 - Always have a default StorageClass, otherwise PVCs that don’t specify a specific class will fail.
 - Give StorageClasses meaningful names.

### Limiting Storage Resource Consumption
It is advised to place limits on container usage of storage, to reflect the amount of storage actually available in the local data center, or the budget available for cloud storage resources.  

There are two main ways to limit storage consumption by containers:
 - Resource Quotas - limits the amount of resources, including storage, CPU and memory, that can be used by all containers within a Kubernetes namespace.
 - StorageClasses - a StorageClass can limit the amount of storage provisioned to containers in response to a PVC.

### Resource Requests and Limits
Kubernetes provides resource requests that help you manage resource consumption allowed for individual containers.
A resource limit can be specified for temporary storage. Setting resource requests and limits can help prevent containers from being constrained by resource scarcity on container hosts, or taking up too many resources unexpectedly.  
Use this command to ensure that all containers in a pod have resource requests and limits:

`kubectl describe pod -n {your_namespace} {your_pod}` 