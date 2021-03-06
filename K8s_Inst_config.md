# K8s Installation and Configuration using Kubeadm

## On Ubuntu 20.04
### Prechecks
1. Check if cluster is already running  
   `which kubeadm`
2. Remove any pre-installed Docker/K8s packages  
   Unhold the packages  
   `sudo apt-mark unhold docker-ce docker-ce-cli kubectl kubeadm kubelet`  
   Remove the packages  
   `sudo apt-get remove -y docker docker.io containerd runc docker-ce docker-ce-cli containerd.io kubeadm kubectl kubelet`  
   Remove unused dependencies  
   `sudo apt autoremove -y`  
   
   > The autoremove option removes packages that were automatically installed because some other package required them but, with those other packages removed, they are no longer needed.
     The packages to be removed are often called "unused dependencies".
3. Remove Docker and k8s files if exists
   ```sh
   cd /var/lib 
   sudo rm -r docker  
   sudo rm -r kubelet
   cd /etc/cni
   sudo rm -r net.d
   ```
4. Disable swap memory  
   `Temporarily disable swap memory`
   ```sh
   sudo swapoff -a # Temporarily disable swap memory 
   swapon -s # Check swap usage
   free -m
   ```
   ` Permanantly disable swap memory`
   ```sh
   # Comment out the swap entry in /etc/fstab
   sudo vi /etc/fstab
   #/swap.img      none    swap    sw      0       0
   ```
5. Enable ip tables - on Ubuntu20
   ```sh
   echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf  
   #Set a value in the sysctl file , to allow proper network settings for Kubernetes on all the servers.
   sudo sysctl -p #To make the changes take immediate effect for the iptables
   ```
### Installation
6. Install Docker  
   ```sh
   sudo apt-get update
   sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
   ```
   Add Dockers official GPG Key
   ```sh
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   sudo apt-get update
   ```
   Install Docker  
   `sudo apt-get install -y docker-ce docker-ce-cli containerd.io --allow-downgrades`
   
   > to Install specific version  
   > sudo apt-get install -y docker-ce=_DocerVersion_ docker-ce-cli=_DocerVersion_ containerd.io --allow-downgrades

7. Install K8s 
   ```sh
   sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list  
   sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -  
   sudo apt-get update  
   sudo apt-get install -y kubeadm kubelet kubectl
   ```
   set lock or hold the k8s and docker packages
   ```sh
   sudo apt-mark hold kubelet kubeadm kubectl docker-ce docker-ce-cli
   ```
   > To Install specific version, check for avaialble versions  
   > ```sh
   > apt-cache madison kubeadm    
   > apt-cache madison kubectl  
   > apt-cache madison kubelet 
   > ```
   > Install specific version
   > ```sh
   > kubever="1.19.1-00"  
   > sudo apt-get install -y kubeadm=$kubever kubelet=$kubever kubectl=$kubever
   > ```
### Initialize the cluster
8. On master node  
   **Initialize the master node**
   ```sh
   sudo kubeadm init --ignore-preflight-errors=all  
   #Copy your join command and keep it safe.
   #Below is an example
   #kubeadm join 10.128.0.16443 --token swi0ci.jq9l75eg8lvpxz9i-discovery-token-ca-cert-hash sha256:2c3cdfa898334b0dfc0f73bbccb998d03f61252ee50f0405c85ba735ff90b5e2
   ```
   > Find the kubeadm join token later  
   > `kubeadm token create --print-join-command --ttl=0`
   
   **set up k8s cluster access**
   ```sh
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```
9. Install network plugin (ex: Flannel)  
   `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`
10. Join the worker nodes using kubeadm join command
   > #kubeadm join 10.128.0.16443 --token swi0ci.jq9l75eg8lvpxz9i-discovery-token-ca-cert-hash sha256:2c3cdfa898334b0dfc0f73bbccb998d03f61252ee50f0405c85ba735ff90b5e2

# K8s deployment on on-premises
## Cluster infrastructure
### Check list
```
1. Control plane high availability
2. worked node high availability
3. Shared storage management
4. observability stack (Cluster monitoring)
```

### Control plane HA
- Master components or control pane components with HA
- etcd should be on two different nodegroups

### Worked nodes HA
**on public cloud**
worker nodes has to be deployed on worker node groups
i.e in auto scaling (AS) groups in different Availability zones (AZ)

### Shared storage management
Shared storage management (EBS and EFS) for Statefulset applications.

### Observability stack
- prometheus and grafana
- ELK

## Cluster services
```
1. Cluster access
2. PSP (pod security policy)
3. custom policies and rules
4. custom DNS (not core DNS)
5. Restricted network policies
```

### Cluster access
- RBAC (Role Based Access Control) or ABAC (Attribute based access control)
- AWS IAM or LDAP
- PSP (Pod security policy)
    - Never deploy a privileged pod into kubesystem name space

### Custom policies and rules
ex:
- Images should be only from private repository
- every name space should have labels
- every pod should have resource limits

### Custom DNS
k8s comes with core DNS, but custom DNS is always recommended.
in case managed k8s (i.e EKS/AKS/GKE) no need to have self managed DNS or custom DNS.

### Restricted network policies
By default k8s allows all traffic across pods with in the cluster which is not recommended, so create a deny all policy and allow only required traffic.


### Security checks
- kube-scan, is for configuration scanning
- kube-bench , is for security bench marks we can check other standards


## Backup and Restore
- CP snap shots
- etcd-DB backup

# Apps and Deployments
- Image quality and vulnerability scanning
- Ingress controller (popular nginx ingress controller)

## Observability
- built in observability is through readyness probe and liveness probe
- third party tools are ELK, Datadog, app dynamics
-  