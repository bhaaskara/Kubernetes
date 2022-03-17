
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