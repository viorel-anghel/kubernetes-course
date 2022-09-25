# Kustomize intro

Helm is really useful when installing open source software and using charts created by other developers, but
the process of creating your own charts is quite tedious.

But let us talk about the *templating* problem
- kubernetes yaml manifests are static files: no variables, no substitutions, nothing but plain values
- when you want to make small changes for deploying in different environments, you will need to edit them. for example
    - change the namespace
    - change the replicas number in a deployment
- you can resort to classical Unix tools and do search and replaces over multiple files with `sed`
- you can create your own Helm charts
- you can use Kustomize

## Kustomize example

The typical use for Kustomize is to have some standard yaml files and to try to deploy them on different environments, let's say you have different specifications for *prod* and *dev* environment.

Check the files here: [kustomize example](kustomize-example).

Kustomize can be used as a standalone tool or directly with `kubectl`:

```
kustomize build dev
kubectl apply -k dev 
```

Official site at https://kustomize.io/ 



