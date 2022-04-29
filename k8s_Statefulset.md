# Kubernetes statefulset
https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

## What is statefulset
 - statefulset is a workload API object used to manage stateful applications.
 - manages the deployment and scaling of set of pods, and provides guarantees about the ordering and uniqueness of these pods.
 - Like a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.
 - Although individual Pods in a StatefulSet are susceptible to failure, the persistent Pod identifiers make it easier to match existing volumes to the new Pods that replace any that have failed.

***Statefulsets require a headless service and we need to create it.***

## When to use statefulsets
-   Stable, unique network identifiers.
-   Stable, persistent storage.
-   Ordered, graceful deployment and scaling.
-   Ordered, automated rolling updates.

## statefulset limitations
 - The storage for a given Pod must either be provisioned by a PersistentVolume Provisioner based on the requested storage class, or pre-provisioned by an admin 
 - Deleting and/or scaling a Statefulset down will not delete the volumes associated with the StatefulSet 
 - StatefulSets currently require Headless Service to be responsible for the network identity of the Pods 
 - StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted 
 - When using Rolling Updates with the default Pod Management Policy (OrderedReady), it's possible to get into a broken state that requires [manual intervention to repair](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#forced-rollback).

> Deleting a statefulset does not guarantee the deletion of the pods or the order in which they get deleted/terminated.
> To achieve a graceful ordered termination Scale down a statefulset to 0, before deleting it.

