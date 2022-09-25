# Kubernetes operators intro

## Resources
- Almost everything in Kubernetes is materialized by a resource
- Resources have a type (or "kind")
- We can see existing types with `kubectl api-resources`
- We can list resources of a given type with `kubectl get <type>`

## Custom resources
- We can create new types with Custom Resource Definitions (CRDs)
- CRDs are created dynamically (without restarting the API server)
- CRDs themselves are resources:
  - we can create a new type with kubectl create and some YAML
  - we can see all our custom types with `kubectl get crds`
- After we create a CRD, the new type works just like built-in types

## CRD Examples
- Representing composite resources: master-slave databases, etc…
- Representing external resources: virtual machines, object store buckets, domain names
- Representing configuration for controllers and operators: custom Ingress resources, certificate issuers, backups
- Alternate representations of other objects; services and service instances: encrypted secret, git endpoints

## Operators
- Operators are software extensions to Kubernetes 
- that uses CRDs to manage applications and their components

The Operator pattern aims to capture the key aim of a human operator who is managing a service or set of services
- Think of a master-slave database
- What is a human operator doing when the master is down?
- Can that be automated?

More info: 
- https://kubernetes.io/docs/concepts/extend-kubernetes/operator/ 
- https://operatorhub.io/ 


## Example - Zalando Postgres Operator

read https://github.com/zalando/postgres-operator

```
# add repo for postgres-operator
helm repo add postgres-operator-charts https://opensource.zalando.com/postgres-operator/charts/postgres-operator

# install the postgres-operator
helm install postgres-operator postgres-operator-charts/postgres-operator
```

create a file
```
# minimal-postgres-manifest.yaml
apiVersion: "acid.zalan.do/v1"
kind: postgresql
metadata:
  name: acid-minimal-cluster
  namespace: default
spec:
  teamId: "acid"
  volume:
    size: 1Gi
  numberOfInstances: 2
  users:
    zalando:  # database owner
    - superuser
    - createdb
    foo_user: []  # role for application foo
  databases:
    foo: zalando  # dbname: owner
  preparedDatabases:
    bar: {}
  postgresql:
    version: "14"
```

```
kubectl create -f minimal-postgres-manifest.yaml

# check the deployed cluster
kubectl get postgresql

# check created database pods
kubectl get pods -l application=spilo -L spilo-role
# check created service resources
kubectl get svc -l application=spilo -L spilo-role

# accessing the db
export PGPASSWORD=$(kubectl get secret postgres.acid-minimal-cluster.credentials -o 'jsonpath={.data.password}' | base64 -d)
export PGSSLMODE=require
psql -U postgres
```
