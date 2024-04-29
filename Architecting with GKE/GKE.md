## GKE
Google Kubenetes engine is a managed service hosted on GCP.
It designed to deploy, manage and scale K8s.

- GKE is fully managed
- Its OS are container optimized
- GKE autopilot manages cluster configuration
- GKE auto upgrade ensures clusters have the latest stable version of Kubernetes
- GKE auto repair repairs unhealthy nodes
- It does a periodic health checks and unhealthy nodes are drained and recreated
- It has cloud build integration
- It has IAM integration
- It has cloud monitoring integration
- VPC integration
- The Google Cloud console provides insights into GKE clusters and their resources

## POD
Pod is the foundational building block of the standard k8s.
Pod is the smallest deployable object.
Every running container in a k8s is in pod.
A pod can have one or more containers.
When there are more containers they are tightly coupled and share the resources like network and storage.
K8s assign a unique IP address to the pod and the containers in the pod share the network namespace including the IP address.
They can communicate using local host, 127.0.0.1



