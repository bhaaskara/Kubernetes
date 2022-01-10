# Kubernetes Secrets
The text or values in secrets is encoded using base64.
Due to this the entire text is not logged any where, i.e logs or debug info.

Secrets can be passed as env variable to the pods
or the text file can be stored on a volume and be mounted.

## Types of secrets
Generic
Certificates
and etc..

## Create a secret from a file
`kubectl create secret generic <mysecret> --from-file=./username.txt --from-file=./password.txt`

Create a secret using literals
`kubectl create secret generic <myliteral> --from-literal=username=user1 --from-literal=password='pwd@1'`

Create a secret with yaml
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

Get/list the secrets
`kubectl get secrets`

Describe the secret
`kubectl describe secrets/<mysecret>`

> base64 encoding: `echo -n 'string' | base64`
   base64 decoding: `echo "encodedstring" | base64 --decode`
