# Important files in K8s

| Dir Location | Contains |
| ------------ | ---------- |
| /etc/kubernetes/manifests | k8s master/control plane components pod definitions, on master node |
| /etc/kubernetes/pki | All certificates |
| $HOME/.kube/config | kube config file to communicate with api server.<br/> *without this file kubectl | can't communicate with api server*  
| /var/log/containers | k8s container logs |

# K8s Commands
## Deployments
Usage | Command
------- | ------
Create deployment | `kubectl create deployment <deploy_name> --image=<image>`
List | `kubectl get deployments`

