# K8s Installation and Configuration using Kubeadm

## On Ubuntu 20
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
   ```
   $cd /var/lib 
   $sudo rm -r docker  
   $sudo rm -r kubelet
   $cd /etc/cni
   $sudo rm -r net.d
   ```
4. Install Docker  
   ```
   $sudo apt-get update
   $sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
   ```
   Add Dockers official GPG Key
   ```
   $sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   $sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   $sudo apt-get update
   ```
   Install Docker  
   `$sudo apt-get install -y docker-ce docker-ce-cli containerd.io --allow-downgrades`
   
   > to Install specific version  
   > $sudo apt-get install -y docker-ce=<DocerVersion> docker-ce-cli=<DocerVersion> containerd.io --allow-downgrades

5. Install K8s 
   ```
   sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list  
   sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -  
   sudo apt-get update  
   sudo apt-get install -y kubeadm kubelet kubectl
   ```
   set lock or hold the k8s and docker packages
   ```
   sudo apt-mark hold kubelet kubeadm kubectl docker-ce docker-ce-cli
   ```
  > To Install specific version, check for avaialble versions  
  > ```
  > apt-cache madison kubeadm    
  > apt-cache madison kubectl  
  > apt-cache madison kubelet 
  > ```
  > Install specific version
  > ```
  > kubever="1.19.1-00"  
  > sudo apt-get install -y kubeadm=$kubever kubelet=$kubever kubectl=$kubever
  > ```

