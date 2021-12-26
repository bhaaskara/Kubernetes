# Helm Cheatsheet

## Helm General Commands
Usage | Command
----- | -------
Helm version | `helm version`

## Helm hub/repo
Usage | Command
----- | -------
search hub | `helm search hub`
search for a keyword | `helm search hub mysql`
Add a repo | `helm repo add stable https://charts.helm.sh/stable`
Update a repo | `helm repo update`
List repos | `helm repo list`
Search in a repo | `helm search repo stable`
Search for mysql in stable repo | `helm seach repo stable/mysql`
list all the available versions | `helm seach repo -l stable/mysql`
Push a chart to repo | `helm push <chart dir> <repo name>` #need push plugin
Remove a repo | `helm repo remove <repo nmae>`

## Helm Charts
Usage | Command
----- | -------
Pull a chart | `helm pull chartrepo/chartname`
Create a chart | `helm create mychart`
Install a chart</br>(from local chart dir ) | `helm install helm-demo-configmap ./mychart`
Install a chart from repo | `helm install stable/mysql --generate-name`
Install a chart with name | `helm install myairflow stable/airflow`
Dry run installing chart | `helm install release1 --debug --dry-run ./chart1`
Uninstall a chart | `helm uninstall myairflow`

## Helm Release/Deployment
Usage | Command
----- | -------
List releases | `helm ls`
Dry run a release | `helm install release1 --debug --dry-run ./chart1`
get the manifest from a release | `helm get manifest release1`
Release status | `helm status <releasename>`
List all resources of a releases | `helm get all <release name>`
Install a chart | `helm install <release name> <chartrepo/chartname>`
Upgrade release | `helm upgrade <release name> <chartrepo/chartname>`
Rollback release | `helm rollback <release name> <revision number>`
Release history | `helm history <release name>`

## Helm Plugins
Usage | Command
----- | -------
List the plugins | `helm plugin list`
Install plugin | `helm plugin install https://github.com/chartmuseum/helm-push.git`
