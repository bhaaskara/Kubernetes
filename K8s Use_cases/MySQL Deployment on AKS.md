# MySQL deployment on K8s AKS
# Using Azure disks

We are going to create a MySQL Database with persistence storage using **Azure Disks**

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

2. Create PVC
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

3. Create configmap for DB creation
```sh
# It creates DB if it doesnt exists
apiVersion: v1
kind: ConfigMap
metadata:
 name: usermanagement-dbcreation-script
data:
 mysql_usermgmt.sql: |-
  DROP DATABASE IF EXISTS webappdb;
  CREATE DATABASE webappdb;
```

4. Create mysql deployment
```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec: 
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate 
  template: 
    metadata: 
      labels: 
        app: mysql
    spec: 
      containers:
        - name: mysql
          image: mysql:5.6
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: dbpassword11
          ports:
            - containerPort: 3306
              name: mysql    
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql    
            - name: usermanagement-dbcreation-script
              mountPath: /docker-entrypoint-initdb.d #https://hub.docker.com/_/mysql Refer Initializing a fresh instance                                            
      volumes: 
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: azure-managed-disk-pvc
        - name: usermanagement-dbcreation-script
          configMap:
            name: usermanagement-dbcreation-script
```

Note: Create all these resources using `kubectl apply -f <folder>/` 

5. Connect to mysql
```
# Connect to MYSQL Database
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -pdbpassword11

# Verify usermgmt schema got created which we provided in ConfigMap
mysql> show schemas;
```

6. Cleanup
```
kubectl delete -f kube-manifests/
```

Note: even after SC and PVC deleted the PV exists due to `retain`policy.
To delete the disk goto azure portal and delete.

When you delete pv the k8s object for pv gets deleted but not the actual disk in azure, due to retain policy.

## Azure AKS references
-   [https://docs.microsoft.com/en-us/azure/aks/concepts-storage](https://docs.microsoft.com/en-us/azure/aks/concepts-storage)
-   [https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-storage](https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-storage)
-   [https://docs.microsoft.com/en-us/azure/aks/azure-disks-dynamic-pv](https://docs.microsoft.com/en-us/azure/aks/azure-disks-dynamic-pv)
-   [https://kubernetes.io/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
-   [https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-disk](https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-disk)
-   [https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#storageclass-v1-storage-k8s-io](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#storageclass-v1-storage-k8s-io)

7. Create clusterIP service to expose the DB
```sh
apiVersion: v1
kind: Service
metadata: 
  name: mysql
spec:
  selector:
    app: mysql 
  ports: 
    - port: 3306  
  clusterIP: None # This means we are going to use Pod IP  
```

8. Deploy user application to use mysql DB
```sh
apiVersion: apps/v1
kind: Deployment 
metadata:
  name: usermgmt-webapp
  labels:
    app: usermgmt-webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: usermgmt-webapp
  template:  
    metadata:
      labels: 
        app: usermgmt-webapp
    spec:
      initContainers:
        - name: init-db
          image: busybox:1.31
          command: ['sh', '-c', 'echo -e "Checking for the availability of MySQL Server deployment"; while ! nc -z mysql 3306; do sleep 1; printf "-"; done; echo -e "  >> MySQL DB Server has started";']      
      containers:
        - name: usermgmt-webapp
          image: stacksimplify/kube-usermgmt-webapp:1.0.0-MySQLDB
          imagePullPolicy: Always
          ports: 
            - containerPort: 8080           
          env:
            - name: DB_HOSTNAME
              value: "mysql"            
            - name: DB_PORT
              value: "3306"            
            - name: DB_NAME
              value: "webappdb"            
            - name: DB_USERNAME
              value: "root"            
            - name: DB_PASSWORD
              value: "dbpassword11"        
```

9. Create service for user web app
```sh
apiVersion: v1
kind: Service
metadata:
  name: usermgmt-webapp-service
  labels: 
    app: usermgmt-webapp
spec: 
  type: LoadBalancer
  selector: 
    app: usermgmt-webapp
  ports: 
    - port: 80
      targetPort: 8080

```

Access application
```
# List Services
kubectl get svc

# Access Application
http://<External-IP-from-get-service-output>
Username: admin101
Password: password101
```

**Note: This is for learning purpose and Drawbacks of it are**
- Complex setup to achieve HA, need Statefulsets
- Complex Master-Master MySQL setup
- Complex Master-slave MySQL setup
- No Automatic Backup & Recovery
- No Auto-upgrade to MySQL
- Logging, Monitoring needs custom sctipts

So its recommended to use Azure MYSQL managed db.