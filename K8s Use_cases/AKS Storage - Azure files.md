Ref: https://github.com/bhaaskara/azure-aks-kubernetes-masterclass/tree/master/08-Azure-Files-for-AKS-Storage

![](Pasted%20image%2020220710000128.png)

**Register for Microsoft.Storage Resource Provider for our Azure Subscription**

In Azure Portal, Go to Billing -> Subscription -> Respective Subscription you are using -> Resource Providers -> Microsoft.storage -> Click Register

![](https://img-c.udemycdn.com/redactor/raw/article_lecture/2022-02-08_09-41-17-1b7f45f9c8e411e136c1c42c13579d8e.png)

  

For **additional reference** about how to register for Resource Providers for our Subscription, please use the below link

https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types


### Custom Storage Class
-   We will define our own custom storage class with desired permissions
    -   Standard_LRS - standard locally redundant storage (LRS)
    -   Standard_GRS - standard geo-redundant storage (GRS)
    -   Standard_ZRS - standard zone redundant storage (ZRS)
    -   Standard_RAGRS - standard read-access geo-redundant storage (RA-GRS)
    -   Premium_LRS - premium locally redundant storage (LRS)
        **Note:** Azure Files support premium storage in AKS clusters that run Kubernetes 1.13 or higher, minimum premium file share is 100GB

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: my-azurefile-sc
provisioner: kubernetes.io/azure-file
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict
parameters:
  skuName: Standard_LRS
```
**Note:** Instead of creating custom storage class, we can use default SC provided by azure.

## PVC
```sh
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-azurefile-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: my-azurefile-sc
  resources:
    requests:
      storage: 5Gi
```

## NGINX deployment
```sh
apiVersion: apps/v1
kind: Deployment 
metadata:
  name: azure-files-nginx-deployment
  labels:
    app: azure-files-nginx-app
spec:
  replicas: 4
  selector:
    matchLabels:
      app: azure-files-nginx-app
  template:  
    metadata:
      labels: 
        app: azure-files-nginx-app
    spec:
      containers:
        - name: azure-files-nginx-app
          image: stacksimplify/kube-nginxapp1:1.0.0
          imagePullPolicy: Always
          ports: 
            - containerPort: 80         
          volumeMounts:
            - name: my-azurefile-volume
              mountPath: "/usr/share/nginx/html/app1"
      volumes:
        - name: my-azurefile-volume
          persistentVolumeClaim:
            claimName: my-azurefile-pvc  
```

## Service to access NGINX app
```
apiVersion: v1
kind: Service
metadata:
  name: azure-files-nginx-service
  labels: 
    app: azure-files-nginx-app
spec:
  type: LoadBalancer
  selector:
    app: azure-files-nginx-app
  ports: 
    - port: 80
      targetPort: 80
```

## Deploy and access NGINX app
```
# Deploy
kubectl apply -f kube-manifests-v1/

# Verify SC, PVC, PV
kubectl get sc, pvc, pv

# Verify Pod
kubectl get pods
kubectl describe pod <pod-name>

# Get Load Balancer Public IP
kubectl get svc

# Access Application
http://<External-IP-from-get-service-output>
http://<External-IP-from-get-service-output>/app1/index.html
```

## Upload Nginx Files to Azure File Share

-   Go to Storage Accounts
-   Select and Open storage account under resoure group **mc_aks-rg1_aksdemo1_eastus**
-   In **Overview**, go to **File Shares**
-   Open File share with name which starts as **kubernetes-dynamic-pv-xxxxxx**
-   Click on **Upload** and upload
    -   file1.html
    -   file2.html

