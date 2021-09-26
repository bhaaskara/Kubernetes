# Get a Shell to a Running Container
```
kubectl exec -it shell-demo -- /bin/bash 
```
Running individual commands in a container
```
kubectl exec shell-demo env
```
examples
```
kubectl exec shell-demo -- ps aux
kubectl exec shell-demo -- ls /
kubectl exec shell-demo -- cat /proc/1/mounts
```
## Opening a shell when a pod has more than one container
```
kubectl exec -i -t my-pod --container main-app -- /bin/bash
```
