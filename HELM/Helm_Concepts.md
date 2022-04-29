**Helm synopsys**
```
Helm is package manager for Kubernetes.
Helm deployes all the reources defined in yaml files under ./<chartname>/templates to k8s cluster.
Helm should be installed and configured on a machine with k8s access through kubectl.
and while deploying it uses the values defined in values.yaml (./<chartname>/values.yaml)
and dependencies be mentioned/defined in chart.yaml (./<chartname>/chart.yaml)
```
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
├── templates                      # upon which helm install action (directory of templates that, when combined with values, will generate valid Kubernetes manifest files.)   
│   ├── NOTES.txt                  # Contents will be displayed upon chart installation     
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

#### Global values
> These values will be available to all the subcharts
`vi values.yaml`
```yml
costCode: 1234
global:
   orgdomain: example.com
```
acccess it using `{{ .Values.global.orgdomain }}`  

### Passing variables through CLI
`helm install release1 ./mychart1 --set costCode=C3245`
> `--set` will have higher precedence over values.yaml file
> This will be useful when u want to set variables particular to that release. 

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

### NOTES.txt
contents will be displayed at the end of successfull chart installtion.
Variables and functions will work in NOTES.txt
`example`
```
Thank you for support {{ .Chart.Name }}.
Your release is named {{ .Release.Name }}.
To learn more about the release, try:

  $ helm status {{ .Release.Name }}
  $ helm get all {{ .Release.Name }}
  $ helm uninstall {{ .Release.Name }}
```
### SubCharts
`Create a sub chart`
```sh
#Goto charts folder under parent chart
cd parentchart/charts
#Create chart
helm create subchart # this creates empty folder structure
```
## Passing values to subchart from parent chart (overriding values in subchart)
`vi mychart/values.yaml`
```yml
subchart:
  dbhostname: prodmyqlnode
```
### by using Global
> These values will be available to all the subcharts
`vi values.yaml`
```yml
costCode: 1234
global:
   orgdomain: example.com
```
acccess it using `{{ .Values.global.orgdomain }}` in sub charts.

# Helm repository management

## Chart Repositories 
- An HTTP server hosting index.yaml file along with chart packages 
- When charts are ready, can be uploaded to the server and shared 
- Multiple charts with dependency and version can be managed 
- Will be managed like source control system in a common location 
- Can be hosed as part Of 
 - Google Compute Cloud Bucket 
 - AWS S3 Bucket 
 - Github pages 
 - Own webserver (chartmuseum) 
 
## Helm Chart repository workflow
![image](https://github.com/bhaaskara/Kubernetes/blob/4adc59a2009132fa7501e5dcd23aca9eb2c010d0/HELM/Helm_Img/Helm_Repo_Workflow.png)

## Chartmuseum repository
`Chartmuseum is your own webserver hosting the charts`

### Install chartmuseum
```sh
curl -LO https://s3.amazonaws.com/chartmuseum/release/latest/bin/linux/amd64/chartmuseum
chmod +x ./chartmuseum
mv ./chartmuseum /usr/local/bin
chartmuseum --version
```
**Run chart museum**
`chartmuseum --debug --port=8080 --storage="local" --storage-local-rootdir="./chartstorage"`  

**Add chartmuseum repo to the local repo list**
```sh
helm repo add chartmuseum http://localhost:8080
helm repo list
helm search repo chartmuseum
```
### Add/Push a chart to chartmuseum
1. Create chart
	`helm create <chartname>`
	> This creates empty chart directory structure, add or modify the files as you need.
2. Package the chart
	`helm package <chartname>`  
3. Add/Push the chart to chartmuseum
	`curl --data-binary "@<chartname>-0.1.0.tgz" http://192.168.0.52:8080/api/charts`  
4. Update and list the chartmuseum repo
	```sh
	helm repo update
	helm repo list
	```
### Maintain chart version
1. Update the chart `Version` and `appVersion` fields in chart.yml
2. Package the chart
	`helm package <chartname>`  
3. Add/Push the chart to chartmuseum
	`curl --data-binary "@<chartname>-0.1.1.tgz" http://192.168.0.52:8080/api/charts`  
4. Update and list the chartmuseum repo
	```sh
	helm repo update
	helm repo list -l <reponame> #will list all the available versions
	```
### Install Helm push plugin
```
helm plugin install https://github.com/chartmuseum/helm-push.git
helm plugin list
```
**Push the chat to repo**  
```
helm create helmpushdemo #Create a chart
helm push helmpushdemo/ mychartmuseumrepo #you can push by using the dir, instead of packaging and copying
```
**List the repo**  
```
helm repo update
helm search repo mychartmuseumrepo
```

## Maintain github as chartrepository
1. Create new repo in github
2. Create a token to upload charts
   `Github -> <repo> -> settings -> Developer settings -> Personal access tokens -> Generate new token`  
   > Select all the scopes except admin scopes
   > copy and save the token, it wont be visible later
3. Intialize local repo and push it to github created repo, ex:helm_git_repo
	```sh
	mkdir helm_git_repo
	cd helm_git_repo
	echo "# helm_repo" >> README.md
	git init
	git add README.md
	git commit -m "first commit"
	git config --global user.email "muthu4all@gmail.com"
	git config --global user.name "Muthukumar"
	git commit -m "first commit"
	git remote add origin https://github.com/<<useraccount>>/helm_git_repo.git
	git push -u origin master
	```
4. Create chart and push it to repo
	```sh
	helm create gitrepotest # This no need to be in git repo dir
	cd helm_git_repo # Get into gitrepo dir to package
	helm package /root/helm_demo/gitrepotest # Provide path to chart dir
	helm repo index .  # Create an index file, in case of chartmuseum its automatically created by chartmuseum
	git add .
	git commit -m "my-repo"
	git push -u origin master
	```
5. Add git chart repo to local repos
	`helm repo add --username <git userid> --password <<acccess token>> my-github-helm-repo 'https://raw.githubusercontent.com/<user>/helm_git_repo/master'

# Helm Chart management
## Upgrade a chart
## Roll back a chart
## Helm Dependency
> Definee the dependent charts in chart.yaml

1. Define the dependency in chart.yaml (./<chatname>/chart.yaml)
```yml
dependencies:
  - name: mariadb
    version: 7.x.x
    repository: https://kubernetes-charts.storage.googleapis.com/
```
2. Build the depedent chat
`helm dependency build ./dependencytest`

3. Deploy the chart

4. Update/Modify the dependencies
`helm dependency update ./dependencytest`

## Helm Chart hooks
Preinstall hooks run after templates are rendered and before any resources are created in K8s. 
Post-install hooks run after all resources are loaded into K8s. 
Pre-delete hooks run before any resources are deleted from K8s. 
Post-delete hooks run after all the release's resources are deleted. 
Pre-upgrade hooks run after templates are rendered and before any resources are loaded into K8s. 
Post-upgrade hooks run after all resources are upgraded. 
Pre-rollback hooks run after templates are rendered and before any resources are rolled back. 
Post-rollback hooks run after all resources are modified. 

### Create chart hooks
1. Create the yaml file under templates. (./<chartname>/templates)
   with annotations
   `preinstall-hook.yaml`
   ```yml
   apiVersion: v1
   kind: Pod
   metadata:
     name: preinstall-hook
     annotations:
       "helm.sh/hook": "pre-install"
   spec:
     containers:
     - name: hook1-container
       image: busybox
       imagePullPolicy: IfNotPresent
       command: ['sh', '-c', 'echo The pre-install hook Pod is running  - preinstall-hook && sleep 10']
     restartPolicy: Never
     terminationGracePeriodSeconds: 0
   ```
   `postinstall-hook.yaml`
   ```yml
   apiVersion: v1
   kind: Pod
   metadata:
     name: postinstall-hook
     annotations:
       "helm.sh/hook": "post-install"
   spec:
     containers:
     - name: hook2-container
       image: busybox
       imagePullPolicy: IfNotPresent
       command: ['sh', '-c', 'echo post-install hook Pod is running - post-install&& sleep 5']
     restartPolicy: Never
     terminationGracePeriodSeconds: 0  
   ```
2. These hooks will get triggered based on the hooks annotation, in this case before and after installing the chart.   

> The pods/jobs created by hooks needs to be deleted/cleaned manually, i.e these wont be deleted on chart uninstall.
> kubectl delete pod/preinstall-hook 
> kubectl delete pod/postinstall-hook

### Create Job as hook
`preinstalljob-hook-job.yaml`
```yml
apiVersion: batch/v1
kind: Job
metadata:   
  name: preinstalljob-hook-job   
  annotations:   
    "helm.sh/hook": "pre-install"   
   
spec:
  template:
    spec:      
      containers:
      - name: pre-installit 
        image: busybox	
        imagePullPolicy: IfNotPresent
        command: ['sh', '-c', 'echo pre-install Job Pod is Running ; sleep 3']
      restartPolicy: OnFailure	
      terminationGracePeriodSeconds: 0
      
  backoffLimit: 3
  completions: 1
  parallelism: 1
```
`postinstalljob-hook-job.yaml`
```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: postinstalljob-hook-job
  annotations:
    "helm.sh/hook": "post-install"
spec:
  template:
    spec:      
      containers:
      - name: post-install
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo post-install Pod is Running ; sleep 6']
    
      restartPolicy: OnFailure
      terminationGracePeriodSeconds: 0
      
  backoffLimit: 3
  completions: 1
  parallelism: 1  
```
> The pods/jobs created by hooks needs to be deleted/cleaned manually, i.e these wont be deleted on chart uninstall.
> kubectl delete job/preinstalljob-hook-job
> kubectl delete job/postinstalljob-hook-job

### Hook execution with weight
```
Define the weight using annotation
Hook with less weight will trigger first
```
`preinstalljob-hook-1.yaml`
```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: preinstalljob-hook-1
  annotations:
    "helm.sh/hook": "pre-install"
    "helm.sh/hook-weight": "-2"

spec:
  template:
    spec:      
      containers:
      - name: pre-install
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo pre-install Job Pod is Running Weight -2 and Sleep 2 ; sleep 2']
    
      restartPolicy: OnFailure
      terminationGracePeriodSeconds: 0
      
  backoffLimit: 3
  completions: 1
  parallelism: 1
```
`preinstalljob-hook-2.yaml`
```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: preinstalljob-hook-2
  annotations:
    "helm.sh/hook": "pre-install"
    "helm.sh/hook-weight": "3"

spec:
  template:
    spec:      
      containers:
      - name: pre-install
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo pre-install Job Pod is Running Weight 3 and Sleep 3 ; sleep 3']
    
      restartPolicy: OnFailure
      terminationGracePeriodSeconds: 0
      
  backoffLimit: 3
  completions: 1
  parallelism: 1
```
`preinstalljob-hook-3.yaml`
```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: preinstalljob-hook-3
  annotations:
    "helm.sh/hook": "pre-install"
    "helm.sh/hook-weight": "5"

spec:
  template:
    spec:      
      containers:
      - name: pre-install
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo pre-install Job Pod is Running Weight 5 and Sleep 5; sleep 5']
    
      restartPolicy: OnFailure
      terminationGracePeriodSeconds: 0
      
  backoffLimit: 3
  completions: 1
  parallelism: 1
```
# Testing and verification
## Syntax check
Validate the chart syntax
`helm lint <chart dir>`

## Hook test
```
Create a test.yaml file under ./<chartname>/templates/test
and annotation should be "hook"
this yaml file will be invoked or deployed when you do
`helm test <release/deployment>`
```

## get and status
`helm get notes <release/deployment>` # Get the release/deployment notes after its deployment  
`helm get values <release/deployment>` # List the user specified values for the release/deployment  
`helm get manifest <release/deployment>` # SHows the manifest file rendered with values (very useful to debug)  
`helm get hooks <release/deployment>` # Get the defined hooks for a release or deployment'  
`helm get all <release/deployment>` # Lists Manifests, Notes and hooks for a release/deployment  
`helm status <release/deployment>` # SHows the name space the resources are deployed their status and the notes  
