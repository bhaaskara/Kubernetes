## Cluster
**`Kubectl get nodes`**
```
NAME      STATUS     ROLES                  AGE   VERSION
master    NotReady   <none>                 90d   v1.23.0
worker1   Ready      control-plane,master   90d   v1.23.0
```
**`kubectl version`**
```
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.0", GitCommit:"ab69524f795c42094a6630298ff53f3c3ebab7f4", GitTreeState:"clean", BuildDate:"2021-12-07T18:16:20Z", GoVersion:"go1.17.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.0", GitCommit:"ab69524f795c42094a6630298ff53f3c3ebab7f4", GitTreeState:"clean", BuildDate:"2021-12-07T18:09:57Z", GoVersion:"go1.17.3", Compiler:"gc", Platform:"linux/amd64"}
```
**`kubectl version --short`**
```
Client Version: v1.23.0
Server Version: v1.23.0
```
**`kubectl version --client`**
```
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.0", GitCommit:"ab69524f795c42094a6630298ff53f3c3ebab7f4", GitTreeState:"clean", BuildDate:"2021-12-07T18:16:20Z", GoVersion:"go1.17.3", Compiler:"gc", Platform:"linux/amd64"}
```

## PODS
**`kubectl  create -f simple_pod  -o name`**
```
pod/test1  #o/p only resource name
```
**`kubectl create -f simple_pod`**
```
pod/test2 created
```