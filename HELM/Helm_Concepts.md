## What is Helm
**A helmsman or “helm” is a person who steers a ship, sailboat, submarine, other types of maritime vessel, or spacecraft.** — Wikipedia  
**The package manager for Kubernetes; Helm is the best way to find, share, and use software built for Kubernetes.** — Helm.sh

## Why do we need Helm
No more maintaining random groups of YAML files (or very long ones) describing pods, replica sets, services, RBAC settings, etc.  
With helm, there is a structure and a convention for a software package that defines a layer of YAML templates and another layer that changes the templates called values.  
Values are injected into templates, thus allowing a separation of configuration, and defines where changes are allowed. This whole package is called a Helm Chart.

# Helm concepts
## Charts
`The chart is a bundle of information necessary to create an instance of a Kubernetes application.`  
Helm uses Charts to pack all the required K8S components for an application to deploy, run and scale. It is also where dependencies are defined, and configurations are updated and maintained.
A chart root has to have only one file named Chart.yaml by convention.
It can optionally (and by default if you use helm create) have a few more components.

**Typical chart file structure:**

mytestchart/                       # Directory with Chart name  
├── Chart.yaml                     # A YAML file containing information about the chart  
├── charts                         # Dir Contains sub charts/Dependencies  
├── templates                      # upon which helm install ation (directory of templates that, when combined with values, will generate valid Kubernetes manifest files.)   
│   ├── NOTES.txt                     
│   ├── _helpers.tpl  
│   ├── deployment.yaml  
│   ├── hpa.yaml  
│   ├── ingress.yaml  
│   ├── service.yaml  
│   ├── serviceaccount.yaml  
│   └── tests  
│       └── test-connection.yaml  
└── values.yaml                   # The default configuration values for this chart  

## Repository
Repositories are where helm charts are held and maintained. In essence, these are a set of templates and configuration values stored as code (sometimes packed as a .tar.gz file).
When you `helm install stable/redis` by default helm reaches out to the repo: [HelmRepo](https://artifacthub.io/)  

## Release
`A release is a running instance of a chart, combined with a specific config.`  
Think of a release as a mechanism to track installed applications on a K8S cluster; when an application is installed by Helm, a release is being created. You can create different installations of a product, e.g., Redis and have two different releases created and tracked in the cluster.
Releases can be tracked with `helm ls`, each would have a “revision” which is the Helm release versioning terminology; if a specific release is updated, e.g., adding more memory to the Redis release, the revision would be incremented. Helm allows rolling back to a particular revision, making it virtually the manager for deployments and production status handler. 

# Chart templates
## Variables
Variables in helm can be defined and used from
- Built in object
- values.yaml file
- through CLI using `--set` keyword
### Built in objects
{{ .Release.Name }}

### values.yaml file (File name starts with small v)
`vi values.yaml`
```yml
costCode: 1234
```
acccess it using `{{ .Values.costCode }}`  
> #*note the caps V while access it using Values.costcode*

### Passing variables through CLI
`helm install release1 ./mychart1 --set costCode=C3245`
> `--set` will have higher precedence over values.yaml file
> This will be useful when u want to set variables peticular to that release. 

## Chart functions
 [Chart Template function](https://masterminds.github.io/sprig/)  
 [Go templates](https://godoc.org/text/template)

`vi values.yaml`
```yml
projectCode: aazzxxyy
 infra:
   zone: a,b,c
   region: us-e
```
Access these variables in templates using
```
Zone: {{ quote .Values.infra.zone }}
Region: {{ quote .Values.infra.region }}
ProjectCode: {{ upper .Values.projectCode }}
```

### Pipeline
```
  var1: {{ .Values.projectCode | upper | quote }}
  now: {{ now | date "2006-01-02" | quote }}
```

### Default value
`contact: {{ .Values.contact | default "1-800-123-0000" | quote }}`
