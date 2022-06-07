Deployment.yml
```yml
apiVersion: apps/vl 
kind: Deployment
metadata: 
  name: php-apache
spec: 
  selector:
    matchLabels:
	  run: php-apache 
  replicas: 1I 
  template: 
    metadata: 
      lables:
	    run: php-apache 
    spec : 
      containers:
	  - name: php-apache 
        image: k8s.gcr.io/hpa-example 
        ports: 
        - containerPort: 80
		resources:
		  requests:
		    cpu: 500m
		  limits:
		    cpu: 1000m
```

HPA.yml
```yml
apiVersion: autoscaling/v1 
kind: HorizontalPodAutoscaler 
metadata: 
  name: php-apache 
  namespace: default 
spec: 
  scaleTargetRef : 
    apiVersion: apps/vl 
    kind: Deployment 
    name: php-apache
  minReplication: 1 
  maxReplication: 10
  targetCPUUtilizationPercentage: 50
```
