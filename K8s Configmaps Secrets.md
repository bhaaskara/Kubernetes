# Kubernetes Configmaps
A ConfigMap is an API object used to store non-confidential data in key-value pairs. [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a [volume](https://kubernetes.io/docs/concepts/storage/volumes/).

A ConfigMap allows you to decouple environment-specific configuration from your [container images](https://kubernetes.io/docs/reference/glossary/?all=true#term-image), so that your applications are easily portable.

> **Caution:** ConfigMap does not provide secrecy or encryption. If the data you want to store are confidential, use a [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) rather than a ConfigMap, or use additional (third party) tools to keep your data private.

A ConfigMap is not designed to hold large chunks of data. The data stored in a ConfigMap cannot exceed 1 MiB. If you need to store settings that are larger than this limit, you may want to consider mounting a volume or use a separate database or file service.

## ConfigMap object[](https://kubernetes.io/docs/concepts/configuration/configmap/#configmap-object)
A ConfigMap is an API [object](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) that lets you store configuration for other objects to use. Unlike most Kubernetes objects that have a `spec`, a ConfigMap has `data` and `binaryData` fields. These fields accept key-value pairs as their values. Both the `data` field and the `binaryData` are optional. The `data` field is designed to contain UTF-8 byte sequences while the `binaryData` field is designed to contain binary data as base64-encoded strings.

The name of a ConfigMap must be a valid [DNS subdomain name](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

Each key under the `data` or the `binaryData` field must consist of alphanumeric characters, `-`, `_` or `.`. The keys stored in `data` must not overlap with the keys in the `binaryData` field.

## ConfigMaps and Pods[](https://kubernetes.io/docs/concepts/configuration/configmap/#configmaps-and-pods)
You can write a Pod `spec` that refers to a ConfigMap and configures the container(s) in that Pod based on the data in the ConfigMap. 
The Pod and the ConfigMap must be in the same [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces).

> **Note:** The `spec` of a [static Pod](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/) cannot refer to a ConfigMap or any other API objects.

```
still need update
https://kubernetes.io/docs/concepts/configuration/configmap/
```

# Kubernetes Secrets
A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key.
Secrets are similar to [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/) but are specifically intended to hold confidential data.
Because Secrets can be created independently of the Pods that use them, there is less risk of the Secret (and its data) being exposed during the workflow of creating, viewing, and editing Pods.
The text or values in secrets is encoded using base64.
Due to this the entire text is not logged any where, i.e logs or debug info.

>**Caution:**
  Kubernetes Secrets are, by default, stored unencrypted in the API server's underlying data store (etcd). Anyone with API access can retrieve or modify a Secret, and so can anyone with access to etcd. Additionally, anyone who is authorized to create a Pod in a namespace can use that access to read any Secret in that namespace; this includes indirect access such as the ability to create a Deployment.
>
> In order to safely use Secrets, take at least the following steps:
> 1.  [Enable Encryption at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) for Secrets.
> 2.  Enable or configure [RBAC rules](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) that restrict reading data in Secrets (including via indirect means).
> 3.  Where appropriate, also use mechanisms such as RBAC to limit which principals are allowed to create new Secrets or replace existing ones.

A Secret can be used with a Pod in three ways:
-   As [files](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod) in a [volume](https://kubernetes.io/docs/concepts/storage/volumes/) mounted on one or more of its containers.
-   As [container environment variable](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables).
-   By the [kubelet when pulling images](https://kubernetes.io/docs/concepts/configuration/secret/#using-imagepullsecrets) for the Pod.

The Kubernetes control plane also uses Secrets; for example, [bootstrap token Secrets](https://kubernetes.io/docs/concepts/configuration/secret/#bootstrap-token-secrets) are a mechanism to help automate node registration.
```
The name of a Secret object must be a valid [DNS subdomain name](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-subdomain-names). 
You can specify the `data` and/or the `stringData` field when creating a configuration file for a Secret. 
The `data` and the `stringData` fields are optional. 
The values for all keys in the `data` field have to be base64-encoded strings. 
If the conversion to base64 string is not desirable, you can choose to specify the `stringData` field instead, which accepts arbitrary strings as values.

The keys of `data` and `stringData` must consist of alphanumeric characters, `-`, `_` or `.`. 
All key-value pairs in the `stringData` field are internally merged into the `data` field. 
If a key appears in both the `data` and the `stringData` field, the value specified in the `stringData` field takes precedence
```

## Types of secrets
When creating a Secret, you can specify its type using the `type` field of a Secret resource, or certain equivalent `kubectl` command line flags (if available). The `type` of a Secret is used to facilitate programmatic handling of different kinds of confidential data.

Builtin Type | Usage
:--- | :--
`Opaque`  | arbitrary user-defined data
`kubernetes.io/service-account-token`  | service account token
`kubernetes.io/dockercfg`  | serialized `~/.dockercfg` file
`kubernetes.io/dockerconfigjson`  | serialized `~/.docker/config.json` file
`kubernetes.io/basic-auth`  | credentials for basic authentication
`kubernetes.io/ssh-auth`  | credentials for SSH authentication
`kubernetes.io/tls`  | data for a TLS client or server
`bootstrap.kubernetes.io/token`  | bootstrap token data

You can define and use your own Secret type by assigning a non-empty string as the `type` value for a Secret object. An empty string is treated as an `Opaque` type.

### Opaque secrets[](https://kubernetes.io/docs/concepts/configuration/secret/#opaque-secrets)
`Opaque` is the default Secret type if omitted from a Secret configuration file. When you create a Secret using `kubectl`, you will use the `generic` subcommand to indicate an `Opaque` Secret type.

```shell
kubectl create secret generic empty-secret
kubectl get secret empty-secret
o/p
----
NAME           TYPE     DATA   AGE
empty-secret   Opaque   0      2m6s
```

### Docker config Secrets[](https://kubernetes.io/docs/concepts/configuration/secret/#docker-config-secrets)
You can use one of the following `type` values to create a Secret to store the credentials for accessing a Docker registry for images.

-   `kubernetes.io/dockercfg`
-   `kubernetes.io/dockerconfigjson`

The `kubernetes.io/dockercfg` type is reserved to store a serialized `~/.dockercfg` which is the legacy format for configuring Docker command line. When using this Secret type, you have to ensure the Secret `data` field contains a `.dockercfg` key whose value is content of a `~/.dockercfg` file encoded in the base64 format.

The `kubernetes.io/dockerconfigjson` type is designed for storing a serialized JSON that follows the same format rules as the `~/.docker/config.json` file which is a new format for `~/.dockercfg`. When using this Secret type, the `data` field of the Secret object must contain a `.dockerconfigjson` key, in which the content for the `~/.docker/config.json` file is provided as a base64 encoded string.

Below is an example for a `kubernetes.io/dockercfg` type of Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-dockercfg
type: kubernetes.io/dockercfg
data:
  .dockercfg: |
        "<base64 encoded ~/.dockercfg file>"
```

When you do not have a Docker config file, or you want to use `kubectl` to create a Docker registry Secret, you can do:

```shell
kubectl create secret docker-registry <secret_name> \
  --docker-username=tiger \
  --docker-password=pass113 \
  --docker-email=tiger@acme.com \
  --docker-server=my-registry.example:5000
```

This command creates a Secret of type `kubernetes.io/dockerconfigjson`

```
https://kubernetes.io/docs/concepts/configuration/secret/
```
## Create a secret from a file
`kubectl create secret generic <mysecret> --from-file=./username.txt --from-file=./password.txt`

Create a secret using literals
`kubectl create secret generic <myliteral> --from-literal=username=user1 --from-literal=password='pwd@1'`

Create a secret with yaml
```
apiVersion: v1
kind: Secret
metadata:
    name: mysecret1
type: opaque
data:
    username: YW!=w4e #base64 encoded data
    password: bXlrwSeq==r
```

Create a Pod to use secret as a volume
```
apiVersion: v1
kind: Pod
metadata:
    name: pod-secret
spec:
    containers:
    - name: mypod
      image: redis
      volumeMounts:
      - name: foo
        mountPath: "/etc/secrets"
        readOnly: true
    volumes:
    - name: foo
      secret:
        secretname: mysecret
```

Create a pod with secrets as env variables
```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
  restartPolicy: Never
```

Get/list the secrets
`kubectl get secrets`

Describe the secret
`kubectl describe secrets/<mysecret>`

> base64 encoding: `echo -n 'string' | base64`  or you can use URL: https://www.base64encode.org
   base64 decoding: `echo "encodedstring" | base64 --decode`

Ref: https://github.com/bhaaskara/azure-aks-kubernetes-masterclass/tree/master/07-Kubernetes-Secrets