# Install prometheus grafana with helm
`helm install prometheus stable/prometheus-operator`

expose the port on deployment
`kubectl port-forward <deployment-prometheus-grafana> 3000`

access grafana on 3000 port or nodeport (if exposed on nodeport)
grafana default credentials: admin/prom-operator

# Install kafka with helm charts
https://dzone.com/articles/how-to-set-up-and-run-kafka-on-kubernetes

