# Helm intro

You will rarely deploy individual objects in Kubernetes. Usually a micro-services application will have
- one or more Deployments
- a Service for every deployment
- an Ingress to provide external access
- optional PVC, configMap, Secret

- Helm is a package manager for Kubernetes
- You can use it to install, customize, upgrade Kubernetes applications
- It uses charts which can be shared in chart repositories
- there are many popular and useful charts
- important links:
  - https://helm.sh/
  - https://artifacthub.io/ -- charts repositories
  - https://github.com/bitnami/charts -- one of the most popular charts provider

---


