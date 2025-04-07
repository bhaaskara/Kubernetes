# Deployment
## What is deployment
A Deployment is a higher-order abstraction that controls deploying and maintaining a set of Pods. Behind the scenes, it uses a `ReplicaSet` to keep the Pods running, but it offers sophisticated logic for deploying, updating, and scaling a set of Pods within a cluster.

A _Deployment_ provides declarative updates for [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) and [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)  
You describe a _desired state_ in a Deployment, and the Deployment [Controller](https://kubernetes.io/docs/concepts/architecture/controller/) changes the actual state to the desired state at a controlled rate.  

## Deployment status

A Deployment enters various states during its lifecycle. It can be [progressing](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#progressing-deployment) while rolling out a new ReplicaSet, it can be [complete](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#complete-deployment), or it can [fail to progress](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#failed-deployment).
- Progressing
- Complete
- Failed
### Progressing Deployment
Kubernetes marks a Deployment as _progressing_ when one of the following tasks is performed:

-   The Deployment creates a new ReplicaSet.
-   The Deployment is scaling up its newest ReplicaSet.
-   The Deployment is scaling down its older ReplicaSet(s).
-   New Pods become ready or available (ready for at least [MinReadySeconds](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#min-ready-seconds)).

You can monitor the progress for a Deployment by using `kubectl rollout status`.

### Complete Deployment
Kubernetes marks a Deployment as _complete_ when it has the following characteristics:

-   All of the replicas associated with the Deployment have been updated to the latest version you've specified, meaning any updates you've requested have been completed.
-   All of the replicas associated with the Deployment are available.
-   No old replicas for the Deployment are running.

You can check if a Deployment has completed by using `kubectl rollout status`

### Failed Deployment
Your Deployment may get stuck trying to deploy its newest ReplicaSet without ever completing. This can occur due to some of the following factors:

-   Insufficient quota
-   Readiness probe failures
-   Image pull errors
-   Insufficient permissions
-   Limit ranges
-   Application runtime misconfiguration

## Operating on a failed deployment
All actions that apply to a complete Deployment also apply to a failed Deployment.   
You can scale it up/down, roll back to a previous revision, or even pause it if you need to apply multiple tweaks in the Deployment Pod template.

## Clean up Policy
You can set `.spec.revisionHistoryLimit` field in a Deployment to specify how many old ReplicaSets for this Deployment you want to retain. The rest will be garbage-collected in the background. By default, it is 10.

> 	**Note:** Explicitly setting this field to 0, will result in cleaning up all the history of your Deployment thus that Deployment will not be able to roll back.
## Use cases
- [Create a Deployment to rollout a ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment).  
    The ReplicaSet creates Pods in the background. Check the status of the rollout to see if it succeeds or not.
- [Declare the new state of the Pods](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment) by updating the PodTemplateSpec of the Deployment. A new ReplicaSet is created and the Deployment manages moving the Pods from the old ReplicaSet to the new one at a controlled rate. Each new ReplicaSet updates the revision of the Deployment.
- [Rollback to an earlier Deployment revision](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment)   
   if the current state of the Deployment is not stable.  
   Each rollback updates the revision of the Deployment.
- [Scale up the Deployment to facilitate more load](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment).
- [Pause the rollout of a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#pausing-and-resuming-a-deployment) to apply multiple fixes to its PodTemplateSpec and then resume it to start a new rollout.
- [Use the status of the Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#deployment-status) as an indicator that a rollout has stuck.
- [Clean up older ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#clean-up-policy) that you don't need anymore.

## Creating a Deployment
Deployment can be created in 3 ways.
- kubectl apply -f [deployment_file]
- kubectl run [deployment-name] --image [image]:[tag] --replicas 3 --labels [key]:[value] --port 8080 --generator deployment/apps.v1 --save.config
- use GKE workloads menu in GCP console

The following is an example of a Deployment.  
It creates a ReplicaSet to bring up three `nginx` Pods:
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

1.  Create the Deployment by running the following command:
    ```shell
    kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
    ```
2.  Run `kubectl get deployments` to check if the Deployment was created.
3. To see the Deployment rollout status, run `kubectl rollout status deployment/nginx-deployment`
4. To see the labels automatically generated for each Pod, run `kubectl get pods --show-labels`

### Writing a Deployment Spec
As with all other Kubernetes configs, a Deployment needs `.apiVersion`, `.kind`, and `.metadata` fields.
A Deployment also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).

### Replicas
`.spec.replicas` is an optional field that specifies the number of desired Pods. It defaults to 1.

### Selector
`.spec.selector` is a required field that specifies a [label selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) for the Pods targeted by this Deployment.

`.spec.selector` must match `.spec.template.metadata.labels`, or it will be rejected by the API.

### Strategy
`.spec.strategy` specifies the strategy used to replace old Pods by new ones. `.spec.strategy.type` can be "Recreate" or "RollingUpdate". "RollingUpdate" is the default value.

#### Recreate Deployment
All existing Pods are killed before new ones are created when `.spec.strategy.type==Recreate`.

#### Rolling Update Deployment
The Deployment updates Pods in a rolling update fashion when `.spec.strategy.type==RollingUpdate`. You can specify `maxUnavailable` and `maxSurge` to control the rolling update process.

##### Max Unavailable
`.spec.strategy.rollingUpdate.maxUnavailable` is an optional field that specifies the maximum number of Pods that can be unavailable during the update process. The value can be an absolute number (for example, 5) or a percentage of desired Pods (for example, 10%). The absolute number is calculated from percentage by rounding down. 
> The value cannot be 0 if `.spec.strategy.rollingUpdate.maxSurge` is 0. The default value is 25%.

##### Max Surge
`.spec.strategy.rollingUpdate.maxSurge` is an optional field that specifies the maximum number of Pods that can be created over the desired number of Pods. The value can be an absolute number (for example, 5) or a percentage of desired Pods (for example, 10%). The value cannot be 0 if `MaxUnavailable` is 0. The absolute number is calculated from the percentage by rounding up. The default value is 25%.

### Revision History Limit
A Deployment's revision history is stored in the ReplicaSets it controls.

`.spec.revisionHistoryLimit` is an optional field that specifies the number of old ReplicaSets to retain to allow rollback. These old ReplicaSets consume resources in `etcd` and crowd the output of `kubectl get rs`. The configuration of each Deployment revision is stored in its ReplicaSets; therefore, once an old ReplicaSet is deleted, you lose the ability to rollback to that revision of Deployment. By default, 10 old ReplicaSets will be kept, however its ideal value depends on the frequency and stability of new Deployments.

> Note: setting this field to zero means that all old ReplicaSets with 0 replicas will be cleaned up. In this case, a new Deployment rollout cannot be undone, since its revision history is cleaned up


## Updating a deployment
> A Deployment's rollout is triggered if and only if the Deployment's Pod template is changed, for example if the labels or container images of the template are updated. Other updates, such as scaling the Deployment, do not trigger a rollout. This means that when you roll back to an earlier revision, only the Deployment's Pod template part is rolled back.

update a deployment
```shell
kubectl set image deployment/<nginx-deployment> nginx=nginx:1.16.1
```
or
```shell
kubectl edit deployment/<nginx-deployment>
```

Check the rollout status
```shell
kubectl rollout status deployment/nginx-deployment
```

## Rollover (aka multiple updates in-flight)
If you update a Deployment while an existing rollout is in progress, the Deployment creates a new ReplicaSet as per the update and start scaling that up, and rolls over the ReplicaSet that it was scaling up previously -- it will add it to its list of old ReplicaSets and start scaling it down.

It does not wait for the previous replicaset to be created before changing course.

## Rolling Back a Deployment
By default, all of the Deployment's rollout history is kept in the system so that you can rollback anytime you want (you can change that by modifying revision history limit).

> A Deployment's rollout is triggered if and only if the Deployment's Pod template is changed, for example if the labels or container images of the template are updated. Other updates, such as scaling the Deployment, do not trigger a rollout. This means that when you roll back to an earlier revision, only the Deployment's Pod template part is rolled back.

> The Deployment controller stops the bad rollout automatically, and stops scaling up the new ReplicaSet. This depends on the rollingUpdate parameters (`maxUnavailable` specifically) that you have specified. Kubernetes by default sets the value to 25%.

1. First, check the revisions of this Deployment:
    ```shell
kubectl rollout history deployment/<nginx-deployment>
```
2. To see the details of each revision, run:
```shell
kubectl rollout history deployment/<nginx-deployment> --revision=2
```
3. Roll back to previous revision
```shell
kubectl rollout undo deployment/nginx-deployment
```
4. Roll back to specific revision
```shell
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

## Scaling a Deployment
```shell
kubectl scale deployment/nginx-deployment --replicas=10
```
Autoscale
>Assuming Horizontal pod autoscaling is enabled in the cluster

```shell
kubectl autoscale deployment/nginx-deployment --min=10 --max=15 --cpu-percent=80
```

## Rolling update

### Proportional scaling
RollingUpdate Deployments support running multiple versions of an application at the same time. When you or an autoscaler scales a RollingUpdate Deployment that is in the middle of a rollout (either in progress or paused), the Deployment controller balances the additional replicas in the existing active ReplicaSets (ReplicaSets with Pods) in order to mitigate risk. This is called _proportional scaling_.

With proportional scaling, you spread the additional replicas across all ReplicaSets. Bigger proportions go to the ReplicaSets with the most replicas and lower proportions go to ReplicaSets with less replicas. Any leftovers are added to the ReplicaSet with the most replicas. ReplicaSets with zero replicas are not scaled up.

## Pausing and Resuming a rollout of a Deployment
When you update a Deployment, or plan to, you can pause rollouts for that Deployment before you trigger one or more updates. When you're ready to apply those changes, you resume rollouts for the Deployment. This approach allows you to apply multiple fixes in between pausing and resuming without triggering unnecessary rollouts.

> Note: You cannot rollback a paused Deployment until you resume it.

```shell
kubectl rollout pause deployment/nginx-deployment
```

## Disruptive updates
In some cases, you may need to update resource fields that cannot be updated once initialized, or you may want to make a recursive change immediately, such as to fix broken pods created by a Deployment.

```shell
kubectl replace -f https://k8s.io/examples/application/nginx/nginx-deployment.yaml --force
```



## Deployment strategy
### Canary deployment
https://phoenixnap.com/kb/kubernetes-canary-deployments

# Deployment commands
kubectl get deployment [deployment_name]
kubectl describe deploymnet [deployment_name]