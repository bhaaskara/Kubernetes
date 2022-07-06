# MySQL deployment on K8s AKS
![](Pasted%20image%2020220705002253.png)

![](Pasted%20image%2020220705002517.png)

1. Create custom storage class in K8s
```sh
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
