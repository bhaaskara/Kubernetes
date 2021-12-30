# Pod
Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers.  
A Pod's contents are always co-located and co-scheduled, and run in a shared context.  
A pod provides a way to set environment variables, mount storage and feed other information into a container.

In Kubernetes, Pods are responsible for running your containers. Every Pod holds at least one container, and controls the execution of that container. 
When the containers exit,the Pod dies too.
 
![image](https://user-images.githubusercontent.com/41310048/137701738-624dc8b4-4c52-46b8-90c4-1c7a44b5d595.png)

If there is more than one container in a pod, they are tightly coupled and share resources including networking and storage.
Kubernetes assigns each Pod a unique IP address. Every container within a Pod shares the network namespace, including IP address and network ports.
Containers within the same Pod can communicate through localhost, 127.0.0.1. A Pod can also specify a set of storage Volumes, to be shared among its containers.

As well as application containers, a Pod can contain `init containers` that run during Pod startup.  
You can also inject *ephemeral containers* for debugging if your cluster offers this.  

> Pods are generally not created directly and are created using workload resources, like Deployments, statefulsets or Daemonsets
> Be careful: Pods are not self-healing.
> Restarting a container in a Pod should not be confused with restarting a Pod. 
> A Pod is not a process, but an environment for running container(s). A Pod persists until it is deleted.

## Container Restart Policy
Flag | Description
---- | ----------
no | Do not automatically restart the container. (Default)
on-failure | Restart the container if it exits due to an error. which manifests as non-zero exit code.
always | Always restart the container if it stops.</br> if its manually stopped, it is restarted only when docker daemon restarts or the container it self manually restarted. 
unless-stopped | similar to always, except when container stopped (manually or otherwise),</br> it is not restarted even after docker daemon restarts. 

## working with Pods
You'll rarely create individual Pods directly in Kubernetes—even singleton Pods. 
This is because Pods are designed as relatively ephemeral, disposable entities. 
When a Pod gets created (directly by you, or indirectly by a controller), the new Pod is scheduled to run on a Node in your cluster. 
The Pod remains on that node until the Pod finishes execution, the Pod object is deleted, the Pod is evicted for lack of resources, or the node fails.

### Pods and controllers
You can use workload resources to create and manage multiple Pods for you.  
A controller for the resource handles replication and rollout and automatic healing in case of Pod failure. 
For example, if a Node fails, a controller notices that Pods on that Node have stopped working and creates a replacement Pod. The scheduler places the replacement Pod onto a healthy Node.

Here are some examples of workload resources that manage one or more Pods:
- Deployment
- StatefulSet
- DaemonSet

### Pod template
Controllers for workload resources create Pods from a pod template and manage those Pods on your behalf.

PodTemplates are specifications for creating Pods, and are included in workload resources such as Deployments, Jobs, and DaemonSets.

Each controller for a workload resource uses the PodTemplate inside the workload object to make actual Pods. The PodTemplate is part of the desired state of whatever workload resource you used to run your app.

If you change the pod template for a workload resource, that resource needs to create replacement Pods that use the updated template.
For example, the StatefulSet controller ensures that the running Pods match the current pod template for each StatefulSet object. If you edit the StatefulSet to change its pod template, the StatefulSet starts to create new Pods based on the updated template. Eventually, all of the old Pods are replaced with new Pods, and the update is complete.


## Pod update and replacement
when the Pod template for a workload resource is changed, the controller creates new Pods based on the updated template instead of updating or patching the existing Pods

Kubernetes doesn't prevent you from managing Pods directly. It is possible to update some fields of a running Pod, in place. 
However, Pod update operations like patch, and replace have some limitations:

- Most of the metadata about a Pod is immutable. 
  For example, you cannot change the 
   - namespace
   - name
   - uid
   - or creationTimestamp fields 
   - the generation field is unique. It only accepts updates that increment the field's current value
- If the metadata.deletionTimestamp is set, no new entry can be added to the metadata.finalizers list.
- Pod updates may not change fields other than 
   - spec.containers[*].image 
   - spec.initContainers[*].image
   - spec.activeDeadlineSeconds
   - spec.tolerations. For spec.tolerations, you can only add new entries.
- When updating the spec.activeDeadlineSeconds field, two types of updates are allowed:
	1. setting the unassigned field to a positive number
	2. updating the field from a positive number to a smaller, non-negative number.

## Resource sharing and communication
Pods enable data sharing and communication among their constituent containers.

### Storage in Pods
A Pod can specify a set of shared storage volumes. All containers in the Pod can access the shared volumes, allowing those containers to share data. Volumes also allow persistent data in a Pod to survive in case one of the containers within needs to be restarted.

### Pod networking
Each Pod is assigned a unique IP address for each address family. Every container in a Pod shares the network namespace, including the IP address and network ports. Inside a Pod the containers that belong to the Pod can communicate with one another using localhost.
The containers in a Pod can also communicate with each other using standard inter-process communications like SystemV semaphores or POSIX shared memory.

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

> Note: The spec of a static Pod cannot refer to other API objects (e.g., ServiceAccount, ConfigMap, Secret, etc).

# Multi container Pods
## Init containers
A Pod can have multiple containers running apps within it, but it can also have one or more init containers, which are run before the app containers are started.
Init containers are exactly like regular containers, except:
 - Init containers always run to completion.
 - Each init container must complete successfully before the next one starts.

If a Pod's init container fails, the kubelet repeatedly restarts that init container until it succeeds. However, if the Pod has a restartPolicy of Never, and an init container fails during startup of that Pod, Kubernetes treats the overall Pod as failed.

**Differences from regular containers**  
 - Init containers support all the fields and features of app containers, including resource limits, volumes, and security settings.
 - Init containers do not support lifecycle, livenessProbe, readinessProbe, or startupProbe because they must run to completion before the Pod can be ready.

If you specify multiple init containers for a Pod, kubelet runs each init container sequentially. Each init container must succeed before the next can run. When all of the init containers have run to completion, kubelet initializes the application containers for the Pod and runs them as usual.

The name of each app and init container in a Pod must be unique; a validation error is thrown for any container sharing a name with another.  
If the Pod restarts, or is restarted, all init containers must execute again.

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

### Pod and Container Fields

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
