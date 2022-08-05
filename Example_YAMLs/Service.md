# Service - Load Balancer
```yml
apiVersion: apps/v1
kind: Service
metadata: 
  name: service-lb 
  labels : 
    app: regapp
spec: 
  type: LoadBalancer
  selector:  
    app: regapp 
  ports:
    - port: 8080
      targetPort: 8080
```

# Service - Multiport
```yml
apiVersion: v1
kind: service
metadata:
    name: mongodb-service
spec:
    selector:
        app: mongodb
    ports:
        - name: mongodb-server   #Mandatory in multiport config
          protocol: TCP
          port: 27017
          targetport: 27017
        - name: mongodb-exporter
          protocol: TCP
          port: 9216
          targetport: 9216 
```
`-name` field is optional in single port service and mandatory in multi port service.
