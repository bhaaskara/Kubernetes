# What is a container
![](Pasted%20image%2020240718224635.png)
**Container**
![](Pasted%20image%2020240718224804.png)

Containers are isolated user spaces for running application code.
![](Pasted%20image%2020241202191916.png)
**Containers are**
- Lightweight because they don't carry a full operating system
- Can be scheduled or integrated tightly with the underlying system
- Can be created and shut down quickly
- Do not boot an entire VM or initialize an operating system for each application


**Why containers are so appealing to developers**
- They're a code-centric way to deliver  high-performing, scalable applications   
- They provide access to reliable underlying hardware and software   
- Code will run successfully regardless if it is on a local machine or in production   
- They make it easier to build applications that use the microservices design pattern

# Container Images
Application and its dependencies are called an image, and container is simply a running instance of an image.

# Docker (Container Run Time)
To build and run container images need software. i.e container run time.
one open source option is Docker.
Docker
	Open source.
	can be used to create and run applications in containers.
	But it can't orchestrate applications at scale.

# Workload isolation
A container has the power to isolate workloads and this ability comes from a combination of several Linux technologies.

![](Pasted%20image%2020241202193824.png)
## Linux process
Each linux process has virtual memory address space separate from all other processes, and linux processes can be rapidly created and destroyed.
## Linux namespaces
Containers use linux name spaces to control what an application can see such as process ID numbers, directory trees, IP address, etc..
Note: Linux name spaces are not the same thing as kubernetes name spaces.
## Linux cgroups
Linux C groups control what an application can use such as its maximum consumption of CPU time, memory, io bandwidth and other resources.
## Union file system
Finally containers use union file system to bundle everything needed into a package. This requires combining applications and their dependencies into a set of clean minimal layers.


# Container Images
![](Pasted%20image%2020241203105408.png)

