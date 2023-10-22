# Certified Kubernetes Administrator
# Core concepts
## Container orchestration
Container orchestration is all about managing the life cycles of containers, especially in   
large, dynamic environments.

Container Orchestration can be used to perform lot of tasks, some of them includes:   
- Provisioning and deployment of containers   
- Scaling up or removing containers to spread application load evenly   
- Movement of containers from one host to another if there is a shortage of resources   
- Load balancing and service discovery between containers   
- Health monitoring of containers and hosts
## Container Orchestration solutions
There are many container orchestration solutions which are available, some of the   
popular ones include:   
- Docker Swarm   
- Kubernetes   
- Apache Mesos   
- Elastic Container Service (AWS ECS)   
There are also various managed container orchestration platforms available like EKS, AKS or GKE.

## Introduction to Kubernetes
Kubernetes (K8s) is an open-source container orchestration engine developed by Google.   
It was originally designed by Google, and is now maintained by the Cloud Native Computing Foundation.

## K8s Architecture
![](Pasted%20image%2020230618175641.png)

## K8s Installation methods
1. Use the Managed Kubernetes Service (AKS,EKS,GKE)
2. Use Minikube
	 Minikube is a tool that makes it easy to run Kubernetes locally.   
     Minikube runs a single-node Kubernetes cluster inside a Virtual Machine (V M) on your laptop for   
    users looking to try out Kubernetes or develop with it day-to-day.
3. Install & Configure Kubernetes Manually (Hard Way)

## Kubectl overview
The Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters.   
You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs.
**Highlevel workflow**
To connect to the Kubernetes Master, there are two important data which kubectl needs:   
1. DNS/ Cluster   
2. Authentication Credentials
![](Pasted%20image%2020230620193812.png)

## K8s Objects
### PODs
A Pod in Kubernetes represents a group of one or more application containers , and some shared resources for those containers.
Containers within a Pod share an IP address and port space, and can find each other via localhost.

![](Pasted%20image%2020230622202022.png)
A Pod always runs on a Node.   
A Node is a worker machine in Kubernetes.   
Each Node is managed by the Master.   
A Node can have multiple pods.

![](Pasted%20image%2020230622202441.png)
**Benifits of Pods**
Many application might have more then one container which are tightly coupled and in   
one-to-one relationship.

**Approach to configure an object**
There are various ways in which we can configure an Kubernetes Object.
- First approach is through the kubectl commands.
- Second approach is through configuration file written in YAML.

# Refs
**Official CKA Exam Blueprint:**
https://github.com/cncf/curriculum
**CKA GitHub Repository Link**
[`https://github.com/zealvora/certified-kubernetes-administrator`](https://github.com/zealvora/certified-kubernetes-administrator)
**Kubectl installation**
[Install Tools | Kubernetes](https://kubernetes.io/docs/tasks/tools/)
