# Concepts
**Imperative configuration**
Achieve desired state of the configuration by issueing commands.
The imperative method is used for quick fixes.
**Declarative configuration**
you define the desired state, and k8s takes cares of maintaining/changing the current state to match the desired state. 

## K8S
Supports stateless apps
Supports stateful apps
Autoscales containerized apps
Allows resource request levels
Allows resource limits
Is extensible, and have lot of plugins
Is open source and portable, which avoids vendor lockin.

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

## K8S Concepts
### K8S Object model
Each item k8s manages is represented by an object, and you can view and change these objects state attributes and state.

K8s objects have two important elements, object spec and status.
Object spec is the desired state defined by user.
Object status is the current state provided by K8S.
### Declarative management
you define the desired state, and k8s takes cares of maintaining/changing the current state to match the desired state. 

## POD
Pod is the foundational building block of the standard k8s.
Pod is the smallest deployable object.
Every running container in a k8s is in pod.
A pod can have one or more containers.
When there are more containers they are tightly coupled and share the resources like network and storage.
K8s assign a unique IP address to the pod and the containers in the pod share the network namespace including the IP address.
They can communicate using local host, 127.0.0.1

LOOKS GOOD NOW

