# Replica set
`Ref:` https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

A Replica Set's purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

The replica set are also known as next generation replication controller. The only difference between replica set and replication controller is the selector types. The replication controller supports equality based selectors whereas the replica set supports equality based as well as set based selectors.

## How a replicaset works
A ReplicaSet is defined with fields, including a selector that specifies how to identify Pods it can acquire, a number of replicas indicating how many Pods it should be maintaining, and a pod template specifying the data of new Pods it should create to meet the number of replicas criteria. A ReplicaSet then fulfills its purpose by creating and deleting Pods as needed to reach the desired number. When a ReplicaSet needs to create new Pods, it uses its Pod template.

A ReplicaSet identifies new Pods to acquire by using its selector. If there is a Pod that has no OwnerReference or the OwnerReference is not a [Controller](https://kubernetes.io/docs/concepts/architecture/controller/) and it matches a ReplicaSet's selector, it will be immediately acquired by said ReplicaSet.

## When to use a replicaset
a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.

## ReplicationController
ReplicaSets are the successors to [_ReplicationControllers_](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/). The two serve the same purpose, and behave similarly, except that a ReplicationController does not support set-based selector requirements as described in the [labels user guide](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors).