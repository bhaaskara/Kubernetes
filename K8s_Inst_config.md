# K8s Installation and Configuration using Kubeadm

## On Ubuntu 20.04
### Prechecks
1. Check if cluster is already running  
   `$which kubeadm`
2. Remove any pre-installed Docker/K8s packages  
   Unhold the packages  
   `$sudo apt-mark unhold docker-ce docker-ce-cli kubectl kubeadm kubelet`  
   Remove the packages  
   `$sudo apt-get remove -y docker docker.io containerd runc docker-ce docker-ce-cli containerd.io kubeadm kubectl kubelet`  
   Remove unused dependencies  
   `$sudo apt autoremove -y`  
   
   > The autoremove option removes packages that were automatically installed because some other package required them but, with those other packages removed, they are no longer needed.
     The packages to be removed are often called "unused dependencies".
3. Remove Docker and k8s files if exists
   ```sh
   $cd /var/lib 
   $sudo rm -r docker  
   $sudo rm -r kubelet
   $cd /etc/cni
   $sudo rm -r net.d
   ```
4. Disable swap memory and reboot vm
   ```sh
   swapon -a
   swapon -s # Check swap usage
   free -m
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
   $sudo apt-get update
   $sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
   ```
   Add Dockers official GPG Key
   ```sh
   $sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   $sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   $sudo apt-get update
   ```
   Install Docker  
   `$sudo apt-get install -y docker-ce docker-ce-cli containerd.io --allow-downgrades`
   
   > to Install specific version  
   > $sudo apt-get install -y docker-ce=<DocerVersion> docker-ce-cli=<DocerVersion> containerd.io --allow-downgrades

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
   ```sh
   sudo kubeadm init --ignore-preflight-errors=all  
   #Copy your join command and keep it safe.
   #Below is an example
   #kubeadm join 10.128.0.16443 --token swi0ci.jq9l75eg8lvpxz9i-discovery-token-ca-cert-hash sha256:2c3cdfa898334b0dfc0f73bbccb998d03f61252ee50f0405c85ba735ff90b5e2
   ```
   > Find the kubeadm join token later  
   > `kubeadm token create --print-join-command --ttl=0`
9. Install network plugin (ex: Flannel)  
   `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`
