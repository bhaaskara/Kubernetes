# Deployment
## What is deployment
A Deployment is a higher-order abstraction that controls deploying and maintaining a set of Pods. Behind the scenes, it uses a `ReplicaSet` to keep the Pods running, but it offers sophisticated logic for deploying, updating, and scaling a set of Pods within a cluster.

- Deployments support rolling updates and rollbacks.
- Roll outs can even be paused.
