
# POD Example YAMLs
## Simple POD
```simple_pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  lables:
    app: myapp
spec:
  containers:
  - name: nginx
    image: nginx  
```

## Pod with generateName (auto name)
```namegen.yaml
apiVersion: v1
kind: Pod
metadata:
  generateName: test-
spec:
  containers:
  - name: nginx
    image: nginx

```
## POD Mutiple containers
## POD Init containers
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```





















