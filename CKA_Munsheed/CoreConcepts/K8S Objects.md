The resources we create in k8s are referred to as objects, like pods, service etc..
Once you create the object, the k8s system will constantly work on to ensure that object exists.

There are various ways in which we can configure an Kubernetes Object.   
- First approach is through the kubectl commands.   
- Second approach is through configuration file written in YAML.
	- Advantages of using config files
		- Integrates well with change review processes. 
		- Provides the source of record on what is live within the Kubernetes cluster. 
		- Easier to troubleshoot changes with version control. 