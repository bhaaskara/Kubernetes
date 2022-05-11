# Important files in K8s

| File / Dir Location | Contains |
| :----------- | :------- |
| /etc/kubernetes/manifests | k8s master/control plane components pod definitions, on master node |
| /etc/kubernetes/pki | All certificates |
| $HOME/.kube/config | kube config file to communicate with api server.<br/> *without this file kubectl can't communicate with api server*  
| /var/log/containers | k8s container logs |

# K8s Commands

# General
Usage | Command
:---- | :------
YAML help | `kubectl explain <pod>`
Syntax check/YAML check | `kubectl create -f sample.yml --dry-run=clinet`
Get the status | `kubectl get -f <file.yml>`
More details | `kubectl describe -f <myapp.yml>`
Pod logs  | `kubectl get logs <pod_name>`
Events | `kubectl get events`
cluster info | `kubectl cluster-info`


## Taint and Tolerances
List the taints
Remove the taints

# Nodes
Usage | Command
:-- | :--
    Get the status | `kubectl get nodes`
# Pods
Usage | Command
:---- | :------
List the pods | `kubectl get pods`
Pods Ip address | `kubectl get pods -o wide`
Pod logs | `kubectl logs dapi-test-pod`
container logs | `kubectl logs <pod_name> -c <container_name>`
Init container logs | `kubectl logs <pod_name> -c <init_container_name>`
Get the pod yaml | `kubectl get pod <pod_name> -o yaml`
Delete a pod | `kubectl delete pod <pod_name>`


### Pod labels
Usage | Command
:---- | :------
List all labels | `kubectl get pods --show-labels`
List pods with label | `kubectl get pods -Lapp -Ltier -Lrole`
 |              | `kubectl get pods --selector="app=redis,role=slave" --show-labels`
 |              | `kubectl get pods --selector="role in (master,slave)" --show-labels`
List pods with lable app    | `kubectl get pods --selector="app" --show-labels`

# Configmaps
Usage | Command
:---- | :------
List Configmaps | `kubectl get configmaps`
Describe Configmap | `kubectl describe configmaps my-config`

Get configmap yaml | `kubectl get configmaps my-config -o yaml`

# Deployments
Usage | Command
:---- | :------
Create deployment | `kubectl create deployment <deploy_name> --image=<image>`
List | `kubectl get deployments`
Details | `kubectl describe deployments`
Edit Deployment | `kubectl edit deployment <deployment_name>`
Update | `kubectl set image deployment/<deploymentname> nginx=<new version> `
 |      | `kubectl edit deplaoyment/<deployment_name>`
Disruptive update | `kubectl replace -f https://k8s.io/examples/application/nginx/nginx-deployment.yaml --force`
Rollout status | `kubectl rollout status deployment/<nginx-deployment>`
 |              | `kubectl get rs`
Rollout history/Revision | `kubectl rollout history deployment/<nginx-deployment>`
Revision details | `kubectl rollout history deployment/<my-app> --revision=2`
Pause a rollout | `kubectl rollout pause deployment/<nginx-deployment>`
Rollback | `kubectl rollout undo deployment/<nginx-deployment>`
Rollback to specific revision | `kubectl rollout undo deployment/<nginx-deployment> --to-revision=2`
events | `kubectl describe <deployment_name>`

# Replica sets
Usage | Command
:---- | :------
List | `kubectl get rs`

# Service
Usage | Command
:---- | :------
Create a service | `kubectl expose pod <pod_name> --name svc1 --port 80 --target-port 80 --type nodeport`
List the services | `kubectl get services`
Describe the service | `kubectl describe svc svc1`
List the backend pods | `kubectl get backends`

# Debugging
Usage | Command
:-- | :--
Pod logs | `kubectl logs <pod_name>`
Events | `kubectl describe pod <pod_name>`
connect to the POD/Container init | `kubectl exec -it <pod name> -- /bin/bash`
