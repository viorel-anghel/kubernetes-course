# Helm intro

You will rarely deploy individual objects in Kubernetes. Usually a micro-services application will have
- one or more Deployments
- a Service for every deployment
- an Ingress to provide external access
- optional PVC, configMap, Secret

- Helm is a package manager for Kubernetes
- You can use it to install, customize, upgrade Kubernetes applications
- It uses *charts* which can be shared in *chart repositories*
- there are many popular and useful charts:
  - postgresql, mysql, cassandra, kafka, redis
  - prometheus, grafana, loki etc
- important links:
  - https://helm.sh/
  - https://artifacthub.io/ -- charts repositories
  - https://github.com/bitnami/charts -- one of the most popular charts provider

---

## Helm versions
Please note we are using Helm version 3 (latest). Version 2 is deprecated and no longer in use. If you find any reference to a tool called Tiller, that is for Helm version 2, i.e. old documentation!.

Also, on Helm 2 charts you will see apiVersion: v1, on Helm 3 - apiVersion: v2.

## Using Helm

- for any helm charts, there are some configurable values along with default values. for example
  - the namespace
  - size for persistence/storage PVC
  - replicas for deployments etc
- those values can be changed with arguments at install time
- or you can create a special file to override the default values for some of the parameters

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

```
# file postgres-values.yml
## see https://github.com/bitnami/charts/tree/master/bitnami/postgresql/

auth:
  postgresPassword: '123456'

primary:
  persistence:
    size: 200Mi
#eof
```

```
# postgres
helm upgrade --install -f postgres-values.yml --create-namespace --namespace default mypg bitnami/postgresql

kubectl get pods,pvc,svc,cm,secret

kubectl exec -ti dummy -- bash
    psql -h  mypg-postgresql -U postgres 
```

Other helm commands:
```
helm ls  # THIS IS PER NAMESPACE
helm uiinstall <SMTH>
helm show chart|readme|values
```

## Creating helm charts
Under this repo https://github.com/viorel-anghel/pgdump-kubernetes/tree/main/helm there is a step by step demo on how to create a Helm chart. 

Basically, 
- you start with some existing yaml manifests 
- you create a `values.yaml` file where you define the chart variables and their default values
- then add your standard yaml manifests files under `templates` and replace some fixed values with variables using `{{ }}`.

