## how to create a sample chart from scratch

1. Create an empty default folder structure for chart named mychart1   
	`helm create mychart1`
2. Create a conifg-map.yml file under templates dir
    > Remove any existing default files under templates dir.
	```yml
	apiVersion: v1
	kind: ConfigMap
	metadata:
	  name: my-chart-configmap
	data:
	  myvalue: "Sample Config Map"
	```
3. Install the chart (creates release)  
	`helm install my-test-release ./mychart1`
	```
	sample o/p
	NAME: my-test-release
	LAST DEPLOYED: Fri Dec 17 10:29:56 2021
	NAMESPACE: default
	STATUS: deployed
	REVISION: 1
	TEST SUITE: None
	```
4. List the releases  
	`helm ls`
	```
	sample o/p
	NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
	my-test-release default         1               2021-12-17 10:29:56.332771457 +0000 UTC deployed        mychart1-0.1.0  1.16.0
	```
