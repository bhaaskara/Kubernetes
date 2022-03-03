# Kubernetes Real Time Project Flow
![[k8s_project_flow.png]](Images/k8s_project_flow.png)

client -> requirements -> choose cloud -> system requirements to create the k8s cluster
-> Developers create container images -> push to registries
-> pull the image through yaml ( either helm, operators, statefulsets or deployments)
-> create infra (terraform to create k8s cluster on AKS,GKE,IKS or AKS)
-> deploy app and DB using ansible or with out ansible (statefulsets or helm)


