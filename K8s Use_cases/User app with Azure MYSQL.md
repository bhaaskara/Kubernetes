![](Pasted%20image%2020220709200245.png)
Here app from AKS can access the external DB using `external name service`.

Ref: https://github.com/bhaaskara/azure-aks-kubernetes-masterclass/tree/master/06-Azure-MySQL-for-AKS-Storage

## Step1: Create Azure MySQL
-   Go to Service **Azure Database for MySQL servers**
-   Click on **Add**
-   **Basics**
-   **Project details**
    -   Subscription: Free Trial
    -   Resource Group: aks-rg1
-   **Server Details**
    -   Server name: akswebappdb (This name is based on availability - in your case it might be something else)
    -   Data source: none
    -   Location: (US) East US
    -   Version: 5.7 (default)
    -   **Compute + Storage**
        -   Pricing Tier: Basic
        -   VCore: 1
        -   Storage: 5GB
        -   Storage Auto Growth: Yes
        -   Backup Retention: 7 days
        -   Locally Redundant: Yes
-   **Administrative Account**
    -   Admin username: dbadmin
    -   Password: Redhat1449
    -   Confirm password: Redhat1449
-   **Review + Create**
-   It will take close to 15 minutes to create the database.

## Update Security Settings for Database

-   Go to **Azure Database for MySQL Servers** -> **akswebappdb**
-   **Settings -> Connection Security**
    -   **Very Important**: Enable **Allow Access to Azure Services**
    -   Update Firewall rules to allow from local desktop (Add current client IP Address)
    -   **SSL Settings**: Disabled
    -   Click on **Save**
-   It will take close to 15 minutes for changes to take place.

```
# Template
mysql --host=mydemoserver.mysql.database.azure.com --user=myadmin@mydemoserver -p

# 
mysql --host=akswebappdb.mysql.database.azure.com --user=dbadmin@akswebappdb -p
```

## Step2: Create Kubernetes externalName service Manifest and Deploy 
-   Create mysql externalName Service
-   **01-MySQL-externalName-Service.yml**
```sh
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: ExternalName
  externalName: akswebappdb.mysql.database.azure.com #Servername of azure db from azure portal
```

-   **Deploy Manifest**
```
kubectl apply -f kube-manifests/01-MySQL-externalName-Service.yml
```

## Step-03: Connect to RDS Database using kubectl and create usermgmt schema/db

```
# Template
kubectl run -it --rm --image=mysql:5.7.22 --restart=Never mysql-client -- mysql -h <AZURE-MYSQ-DB-HOSTNAME> -u <USER_NAME> -p<PASSWORD>

# Replace Host Name of Azure MySQL Database and Username and Password
kubectl run -it --rm --image=mysql:5.7.22 --restart=Never mysql-client -- mysql -h akswebappdb.mysql.database.azure.com -u dbadmin@akswebappdb -pRedhat1449

mysql> show schemas;
mysql> create database webappdb;
mysql> show schemas;
mysql> exit
```

## Step4: User management webapp deployment
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
              value: "mysql"  #ExternalNameService name          
            - name: DB_PORT
              value: "3306"            
            - name: DB_NAME
              value: "webappdb"            
            - name: DB_USERNAME
              value: "dbadmin@akswebappdb"            
            - name: DB_PASSWORD
              value: "Redhat1449"      
```

**User management webapp service**
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

```
# Deploy all Manifests
kubectl apply -f kube-manifests/usermanagementapp.yml
kubectl apply -f kube-manifests/usermanagementappservice.yml

# List Pods
kubectl get pods

# Stream pod logs to verify DB Connection is successful from SpringBoot Application
kubectl logs -f <pod-name>
```

## Step-06: Access Application

```
# Get Public IP
kubectl get svc

# Access Application
http://<External-IP-from-get-service-output>
Username: admin101
Password: password101
```

## Step-07: Clean Up

```
# Delete all Objects created
kubectl delete -f kube-manifests/

# Verify current Kubernetes Objects
kubectl get all

# Delete Azure MySQL Database
```