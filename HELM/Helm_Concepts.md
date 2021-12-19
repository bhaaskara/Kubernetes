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
│   ├── _helpers.tpl               # contains the templates
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

### values.yaml file (File name should starts with small v, and extension should be yaml)
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

## Flow control
### If-else
```yml
{{ if <condition1> }}
#true block
{{ else if <condition> }}
#else if block
{{ else }}
#false block
{{ end }}
```
A condition is evaluated as false if the value is:
- a Boolean false
- a numeric zero
- an empty string
- a nil (empty or null)
- an empty collection (map,slice,tuple,dict,array)



```yml
  #`indentation is very important`
  {{- if eq .Values.infra.region "us-e" }}  #`- is to remove the new line`
  ha: true
  {{- end }}
```
### Defining scope using WITH
> Built in objects can't be accessed with in the scope

`vi values.yaml`
```yml
tags:
  machine: frontdrive
  rack: 4c
  drive: ssd
  vcard: 8g
```
Access in with block
```yml
metadata:
  name: {{ .Release.Name}}-configmap
  labels:
  {{- with .Values.tags }} #`- to remove new line`
    first: {{ .machine }}
    second: {{ .rack }}
    third: {{ .drive }}
  {{- end }}
```
### Loop using Range
`vi values.yaml`
```yml
LangUsed:
  - Python
  - Ruby
  - Java
  - Scala
```
Loop through using range
```yml
LangUsed: |-    # |- defines the variable as list
   {{- range .Values.LangUsed }}
   - {{ . | title | quote }}
   {{- end }}   
```
### Variables
Assig a value and access the variable
```yml
{{- $relname := .Release.Name -}} # assign
{{ $relname }} # use/access
```
`vi values.yaml`
```yml
tags:
  machine: frontdrive
  rack: 4c
  drive: ssd
  vcard: 8g
```
```yml
tags: 
  {{- range $key, $value := .Values.tags }}
  {{ $key }} : {{ $value }}
  {{- end }} 
```
```
$ - here refers to the global context, where as '.' refers to the current context

 labels:
   helm.sh/chart: "{{ $.Chart.Name }}-{{ $.Chart.Version }}"
   app.kubernetes.io/instance: "{{ $.Release.Name }}"
   app.kubernetes.io/version: "{{ $.Chart.AppVersion }}"
  app.kubernetes.io/managed-by: "{{ $.Release.Service }}"
```
### Include content from same file (define)
`vi configmap.yaml` under /chart/templates
```yml
{{- define "mychart.systemlables" }}
  labels:
    drive: ssd
    machine: frontdrive
    rack: 4c
    vcard: 8g
{{- end }}
{{- define "var2" }}
  labels:
    drive: hdd
    machine: frontdrive
    rack: 4c
    vcard: 8g
{{- end }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
  {{- template "mychart.systemlables" }}
  {{- template "var2"}}
data:
  myvalue: "Sample Config Map"
  costcode: {{.Values.costCode }}
  {{- if eq .Values.infra.region "us-e" }}
  ha: true
  {{- end }}
  langused: |-
          {{- range .Values.LangUsed }}
          - {{ . | title | quote }}
          {{- end }}
```
### Include content from helper file (_'_helpers.tpl'_)
#### template
> file name can be anything

vi _'_helpers.tpl'_ (under templates)
```yml
{{- define "mychart.systemlables" }}
  labels:
    drive: ssd
    machine: frontdrive
    rack: 4c
    vcard: 8g
{{- end }}
{{- define "var2" }}
  labels:
    drive: hdd
    machine: frontdrive
    rack: 4c
    vcard: 8g
    app.kubernetes.io/instance: "{{ $.Release.Name }}"
    app.kubernetes.io/version: "{{ $.Chart.AppVersion }}"
    app.kubernetes.io/managed-by: "{{ $.Release.Service }}"
{{- end }}
```
vi configmap.yml (under charts)
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
  {{- template "mychart.systemlables" }}
  {{- template "var2"}}
data:
  myvalue: "Sample Config Map"
  costcode: {{.Values.costCode }}
```
```
use context: $ (global) . (current)
with out the context global builtin objects can't be accessed.
```
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
  {{- template "mychart.systemlables" $ }} #Global context is recommended
  {{- template "var2" .}}
data:
  myvalue: "Sample Config Map"
  costcode: {{.Values.costCode }}
```
#### include
`vi _helpers.tpl`
```yml
{{- define "mychart.version" -}}
app_name: {{ .Chart.Name }}
app_version: "{{ .Chart.Version }}"
{{- end -}}
```
`vi configmap.yml`
```yml
 metadata:
   name: {{ .Release.Name}}-configmap
   labels:
 {{ include "mychart.version" . | indent 4 }}
 data:
   myvalue: "Sample Config Map"
   costCode: {{ .Values.costCode }}
   Zone: {{ quote .Values.infra.zone }}
   Region: {{ quote .Values.infra.region }}
   ProjectCode: {{ upper .Values.projectCode }}
   pipeline: {{ .Values.projectCode | upper | quote }}
   now: {{ now | date "2006-01-02"| quote }}
   contact: {{ .Values.contact | default "1-800-123-0000" | quote }}
   {{- range $key, $value := .Values.tags }}
   {{ $key }}: {{ $value | quote }}
   {{- end }}
{{include "mychart.version" $ | indent 2 }}
```
> Include vs template  
> template
> - the indentation in the source/definition file is used while importing.
> Include  
> - allows more control on indentation and is recommended over template.
