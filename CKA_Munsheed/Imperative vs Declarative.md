# Imperative

# Declarative

# Kubectl apply
kubectl apply command uses three files/objects 
- Object definition file (created by user)
- Last applied configuration (saved as annotations in the K8s object state)
- K8s object live status 
to compare and take a decision to change the object.

![](Pasted%20image%2020250603175227.png)
> Note: the annotations are not created if the object is created in imperative way (kubectl create ).
> never mix both imperative and declarative methods when creating objects.

