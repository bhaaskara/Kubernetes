# DaemonSet
https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#daemonset

Use a [`DaemonSet`](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) instead of a ReplicaSet for Pods that provide a machine-level function, such as machine monitoring or machine logging. These Pods have a lifetime that is tied to a machine lifetime: the Pod needs to be running on the machine before other Pods start, and are safe to terminate when the machine is otherwise ready to be rebooted/shutdown.
