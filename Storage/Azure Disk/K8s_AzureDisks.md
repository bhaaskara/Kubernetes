# Azure Disks
• Azure Disk Storage offers high-performance, highly durable block storage for our mission- and business-critical workloads 
• We can mount these volumes as devices on our Virtual Machines & Container instances. 
• Cost-effective storage 
	• Built-in bursting capabilities to handle unexpected traffic and process batch jobs cost-effectively 
• Unmatched resiliency 
	• 0 percent annual failure rate for consistent enterprise-grade durability 
• Seamless scalability 
	• Dynamic scaling of disk performance on Ultra Disk Storage without disruption 
• Built-in security 
	• Automatic encryption to help protect your data using Microsoft-managed keys or your own 

# Storage Class
```yml
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
```
**volumeBindingMode: Immediate** - implies that the PersistentVolumecreation,
followed with the storage medium (Azure Disk in this case) provisioning is triggered as soon as the PersistentVolumeClaim is created.

**volumeBindingMode: WaitForFirstConsumer** By default, the Immediate mode indicates that volume binding and dynamic provisioning occurs once the PersistentVolumeClaim is created. For storage backends that are topology-constrained and not globally accessible from all Nodes in the cluster PersistentVolumes will be bound or provisioned without knowledge of the Pod's scheduling requirements. This may result in unschedulable Pods.
A cluster administrator can address this issue by specifying the WaitForFirstConsumer
mode which will delay the binding and provisioning of a PersistentVolume until a
Pod using the PersistentVolumeClaim is created. PersistentVolumes will be selected or
provisioned conforming to the topology that is specified by the Pod's scheduling
constraints.

**reclaimPolicy: Delete** With this setting, as soon as a PersistentVolumeClaim is deleted,
it also triggers the removal of the corresponding PersistentVolume along with the Azure Disk.
We will be surprised provided if we intended to retain that data as backup.

**reclaimPolicy: retain** Disk is retained even when PVC is deleted - Recommended Option

Both **reclaimPolicy: Delete** and **volumeBindingMode: Immediate** are default settings

**kind: Managed** When managed used, that disk is persisted for the Lifecycle of the cluster.
If we delete cluster, it will delete the disk

## Azure Disk default storage classes in K8s
![](Pasted%20image%2020220705220507.png)

# PersistentVolumeClaim (PVC)
```yml
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