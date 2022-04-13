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
