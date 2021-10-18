# Pod
Pods are the basic unit for running containers inside kubernetes.  
A pod provides a way to set environment variables, mount storage and feed other information into a container.  

In Kubernetes, Pods are responsible for running your containers. 
Every Pod holds at least one container, and controls the execution of that container. When the containers exit,the Pod dies too.  
![image](https://user-images.githubusercontent.com/41310048/137701738-624dc8b4-4c52-46b8-90c4-1c7a44b5d595.png)

If there is more than one container in a pod, they are tightly coupled and share resources including networking and storage.  
Kubernetes assigns each Pod a unique IP address. Every container within a Pod shares the network namespace, including IP address and network ports.  
Containers within the same Pod can communicate through localhost, 127.0.0.1. A Pod can also specify a set of storage Volumes, to be shared among its containers.  
> Be careful: Pods are not self-healing.

# Environment Variables
When building your application stack to work on Kubernetes, the basic pod configuration is usually done by setting different environment variables. Sometimes you want to configure just a few of them for a particular pod or to define a set of environment variables that can be shared by multiple pods. Later is usually done by creating a ConfigMap as a shared resource. Instead of specifying each environment variable individually we can reference the whole config map.

Set Environment Variables

To set the environment variables, you can use env or envFrom key in the configuration file. The most basic option is to set one or more of them using the simple key:value syntax:

	spec:
	  containers:
	  - env:
		- name: VARIABLE1
		  value: test1
		  
## ConfigMap
It looks okay, but imagine ten or more variables per pod. It would be much better to have a separate configuration file. 
In Kubernetes you can do that by utilizing config maps. Here is an example:

	apiVersion: v1
	kind: ConfigMap
	metadata:
	  name: my-env
	data:
	  VARIABLE1: test1
	  VARIABLE2: test2
	  VARIABLE3: test3

With the above configuration, it is easy for multiple containers to share the same set of environment variables. Then we need to reference the config map file with configMapRef (available from Kubernetes v1.6):

	spec:
	  containers:
	  - envFrom:
		  - configMapRef:
			  name: my-env

In case you don't want for each container to have all environment variables from the config map, you can get specific keys only. For example, get VARIABLE1 in the pod:

	spec:
	  containers:
	  - env:
		- name: VARIABLE1
		  valueFrom:
			configMapKeyRef:
			  name: my-env
			  key: VARIABLE1

## Pod and Container Fields

Also, you can get pod and container fields that are available through Kubernetes API and set them as environment variables. Here is the list of available pod and container fields - replace <CONTAINER_NAME> with your container name to get container fields:

	spec:
	  containers:
	  - env:
		- name: MY_NODE_NAME
		  valueFrom:
			fieldRef:
			  fieldPath: spec.nodeName
		# Kubernetes 1.7+
		- name: MY_NODE_IP
		  valueFrom:
			fieldRef:
			  fieldPath: status.hostIP
		- name: MY_POD_NAME
		  valueFrom:
			fieldRef:
			  fieldPath: metadata.name
		- name: MY_POD_NAMESPACE
		  valueFrom:
			fieldRef:
			  fieldPath: metadata.namespace
		- name: MY_POD_IP
		  valueFrom:
			fieldRef:
			  fieldPath: status.podIP
		- name: MY_POD_SERVICE_ACCOUNT
		  valueFrom:
			fieldRef:
			  fieldPath: spec.serviceAccountName
		# Kubernetes 1.8+
		- name: MY_POD_UID
		  valueFrom:
			fieldRef:
			  fieldPath: metadata.uid
		- name: MY_CPU_REQUEST
		  valueFrom:
			resourceFieldRef:
			  containerName: <CONTAINER_NAME>
			  resource: requests.cpu
		- name: MY_CPU_LIMIT
		  valueFrom:
			resourceFieldRef:
			  containerName: <CONTAINER_NAME>
			  resource: limits.cpu
		- name: MY_MEM_REQUEST
		  valueFrom:
			resourceFieldRef:
			  containerName: <CONTAINER_NAME>
			  resource: requests.memory
		- name: MY_MEM_LIMIT
		  valueFrom:
			resourceFieldRef:
			  containerName: <CONTAINER_NAME>
			  resource: limits.memory
			  
> As we can see, there are a lot of options available in Kubernetes when defining environment variables. You need to pick the right approach. If you want to manage sensitive information like passwords and other secrets, then you should use Secret instead of ConfigMap.

## Static Pod
Static Pods are managed directly by the kubelet daemon on a specific node, without the API server observing them. Unlike Pods that are managed by the control plane (for example, a Deployment). instead, the kubelet watches each static Pod (and restarts it if it fails).
Static Pods are always bound to one Kubelet on a specific node.
The kubelet automatically tries to create a mirror Pod (an object in the API server that tracks a static POD on the node)on the Kubernetes API server for each static Pod. This means that the Pods running on a node are visible on the API server, but cannot be controlled by control plane.  
**The Pod names will be suffixed with the node hostname with a leading hyphen.**

> If you are running clustered Kubernetes and are using static Pods to run a Pod on every node, you should probably be using a DaemonSet instead.

To create a static Pod, create its yaml definition file under `/etc/kubernetes/manifests` and Remove the yaml file to delete the pod.
> the path to the manifests file can be found at `/var/lib/kubelet/config.yaml` Only kubelet manages the static pod, means API server or any of the controller don’t have control on it.  

**Static pod use case**
All the k8s master components run as a static pods on master node.

