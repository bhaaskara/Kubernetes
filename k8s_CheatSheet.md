# Important files in K8s

| Dir Location | Contains |
| :----------- | :------- |
| /etc/kubernetes/manifests | k8s master/control plane components pod definitions, on master node |
| /etc/kubernetes/pki | All certificates |
| $HOME/.kube/config | kube config file to communicate with api server.<br/> *without this file kubectl | can't communicate with api server*  
| /var/log/containers | k8s container logs |

# K8s Commands
## Pods
Usage | Command
:---- | :------
Pods Ip address | `kubectl get pods -o wide`

## Deployments
Usage | Command
:---- | :------
Create deployment | `kubectl create deployment <deploy_name> --image=<image>`
List | `kubectl get deployments`
Deatils | `kubectl describe deployments`
Update | `kubectl set image deployment/<deploymentname> nginx=<new version> `
|      | `kubectl edit deplaoyment/<deployment_name>`
Disruptive update | `kubectl replace -f https://k8s.io/examples/application/nginx/nginx-deployment.yaml --force`
Rollout status | `kubectl rollout status deployment/<nginx-deployment>`
|              | `kubectl get rs`
Rollout history/Revision | `kubectl rollout history deployment/<nginx-deployment>`
Pause a rollout | `kubectl rollout pause deployment/<nginx-deployment>`
Rollback | `kubectl rollout undo deployment/<nginx-deployment>`
Rollback to specific revision | `kubectl rollout undo deployment/<nginx-deployment> --to-revision=2`

## Replica sets
Usage | Command
:---- | :------
List | `kubectl get rs`

## Service
Usage | Command
:---- | :------
List the backend pods | `kubectl get backends`