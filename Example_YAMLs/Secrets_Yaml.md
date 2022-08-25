Get/list the secrets
`kubectl get secrets`

Describe the secret
`kubectl describe secrets/<mysecret>`

# Create a Secret
## Generic Secret
```
apiVersion: v1
kind: Secret
metadata:
    name: mysecret1
type: opaque
data:
    username: YW!=w4e #base64 encoded data
    password: bXlrwSeq==r
```

## Dockercfg type secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-dockercfg
type: kubernetes.io/dockercfg
data:
  .dockercfg: |
        "<base64 encoded ~/.dockercfg file>"
```

## Docker registry secret using `kubectl`
```shell
kubectl create secret docker-registry <secret_name> \
  --docker-username=tiger \
  --docker-password=pass113 \
  --docker-email=tiger@acme.com \
  --docker-server=my-registry.example:5000
```

# Access a secret
Create a Pod to use secret as a volume
```
apiVersion: v1
kind: Pod
metadata:
    name: pod-secret
spec:
    containers:
    - name: mypod
      image: redis
      volumeMounts:
      - name: foo
        mountPath: "/etc/secrets"
        readOnly: true
    volumes:
    - name: foo
      secret:
        secretname: mysecret
```

Create a pod with secrets as env variables
```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
  restartPolicy: Never
```

