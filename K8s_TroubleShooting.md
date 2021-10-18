# Cluster Troublshooting
## Check kubelet status  
`systemctl status kubelet`  
>● kubelet.service - kubelet: The Kubernetes Node Agent  
>   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)  
>  Drop-In: /etc/systemd/system/kubelet.service.d  
>           └─10-kubeadm.conf  
>   **Active: active (running)** since Mon 2021-10-18 12:59:00 UTC; 37min ago  
>     Docs: https://kubernetes.io/docs/home/  
> Main PID: 785 (kubelet)  
>    Tasks: 12 (limit: 1105)  
>   CGroup: /system.slice/kubelet.service  
>           └─785 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.  
>
>**Oct 18 13:36:20 master kubelet[785]: E1018 13:36:20.943484     785 kubelet.go:2407] "Error getting node" err="node \"master\" not found**  
>**Oct 18 13:36:21 master kubelet[785]: E1018 13:36:21.044195     785 kubelet.go:2407] "Error getting node" err="node \"master\" not found**  

Verify the status and for the errors.  
Here error is master not found, either master is not running or not reachable.  
## Check docker status
`systemctl status docker`
>● docke.sevice - Docke Application Containe Engine  
>   Loaded: loaded (/lib/systemd/system/docke.sevice; enabled; vendo peset: enabled)  
>   **Active: active (unning)** since Mon 2021-10-18 12:59:11 UTC; 53min ago  
>     Docs: https://docs.docke.com  
> Main PID: 1068 (docked)  
>    Tasks: 32  
>   CGoup: /system.slice/docke.sevice  
>           └─1068 /us/bin/docked -H fd:// --contained=/un/contained/contained.sock  
>  
>Oct 18 13:28:43 maste docked[1068]: time="2021-10-18T13:28:43.627017991Z" level=info msg="ignoing event" containe=aac9cb7cd155ea9  
>Oct 18 13:30:42 maste docked[1068]: time="2021-10-18T13:30:42.242135699Z" level=info msg="ignoing event" containe=2e961652eab5c1c
## Check APIserver ip / Kube-master ip
check in file $HOME/.kube/config on master node
> server: https://192.168.0.104:6443  
> This should be your master node IP.
