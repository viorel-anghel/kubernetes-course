# CI/CD, GitOps and ArgoCD

## CI/CD
- CI/CD = continous integration (build) / continous deployment

- developers push code in a repository (git, github, gitlab etc)
- a build system monitors the code repository and triggers an automatic build when code is pushed on master branch
  - for example jenkins, bamboo, tekton, argo events etc
- the build continue with tests. if those fails, developers are notified and the process stops
- if the teste pass, the build artifacts are stored, for example new version docker images in a docker registry
- also, the CD process will be prepared and/or triggered
- the CD process will usually automatically deploy the new software version in a dev/staging environment
- and then, on-click, deploy in a production environment

## GitOps
- Git plays a key role in GitOps. 
- Git is used as a *single source of truth* for what infrastructure and applications should look like 
- based on *declarative configuration* (not imperative instructions)
- automated processes are then used to take this configuration and make sure the target environment *actual state* matches the *desired state* (described in the repository)

## ArgoCD
In this whole equation, *ArgoCD* is just a simple brick, the CD / deployment part. It is a new tool but quite popular, it is a very well fit in the Kubernetes culture and also it has a nice web interface.

- ArgoCD is running in a kubernetes cluster
- and has the concept of *application*, which is in turn has
    - SOURCE: a git repository ( != application code repo)
      - kubernetes manifests files (yaml) (and/or helm charts and/or kustomize). basically, how you deploy
    - DEST: a kubernetes cluster 
    - when the source changes, ArgoCD automatically deploy what changed in the destination cluster.


## Install and demo




