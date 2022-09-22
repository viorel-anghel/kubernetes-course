# Kubernetes intro

Start with this presentation: **Kubernetes intro** (gdrive viorel@esol)

## Using Kubernetes with Rancher Desktop

- enable Kubernetes in Rancher Desktop
- run `docker ps` - you will see many (13-14) containers with the name `k8s_...`
- also the file `~/.kube/config` should be created

We plan to use multiple Kubernetes clusters. There are many ways to do this but my prefference is to have a configuration file for each one. So, save that file with another name, then tell `kubectl` to use it with the new name:

```
mv ~/.kube/config ~/.kube/config-rd
export KUBECONFIG=~/.kube/config-rd
echo $KUBECONFIG
kubectl get nodes
```

--- 

## Accessing a remote Kubernetes cluster

We'll also use a Kubernetes cluster started in DigitalOcean. 

I'll share with you the kubeconfig file, save it under `~/.kube/config-do`.

---

## Ways to install an on-premise Kubernetes cluster

- for testing/dev environments: KIND - Kubernetes in Docker
- for production:
  - Kubernetes the hard way
  - kubeadm
  - rke2 from Rancher

---

## Info about nodes

```
kubectl get nodes
kubectl get nodes -o wide
kubectl get nodes -o json   # or yaml
kubectl describe node <name>
```

---

## What is a POD?

- Kubernetes uses the concept of POD as the minimal deployable object. 
- A POD is a group of (usually) one or more containers :
- running together (on the same node)
- sharing resources (RAM, CPU; but also network, volumes)
- pod's containers are created from container images as with Docker

---

## Starting a POD

- many Kubernetes objects (resources) can be created with 
  - an *imperative* command, for example `kubectl run nginx --image=nginx`
  - or with a declarative method

For the declarative method, you will usually create a `yaml manifest` describing all the aspects of the resource and then use the command `kubectl apply -f <manifest.yaml>` to create the resources.

```
# nginx2-pod.yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx2
spec:
  containers:
    - name: nginx
      image: nginx
# eof
```

Similarly, you may delete resources:
- with imperative commands: `kubectl delete pod nginx`
- with declarative commands: `kubectl delete -f <manifest.yaml>`

---

## Info about running pods

```
# wide output will show pod's IP and on which node are allocated:
kubectl get pods -o wide   

# -o yaml will produce an yaml manifest which can be used 
# to edit/recreate the resource
kubectl get pod nginx -o yaml >save.yaml
less save.yaml

# describe will show full info about the resource, including an event log
kubectl describe pod nginx

# showing logs # no need to specify resource type pod, only pods have logs
kubectl logs nginx 
```

## Entering a running pod

As with Docker, we can enter inside a running pod (container) and run commands inside and copy files in out.

```
kubectl exec -ti nginx -- sh
kubectl cp nginx:/etc/nginx/nginx.conf /tmp/nginx.conf
```

---

## Namespaces

- Namespaces allow us to segregate resources
- By default, kubectl will reffer to the namespace `default`
- Then we can use -n / --namespace with almost every kubectl command

```
kubectl get namespaces
kubectl get pods -n kube-system
kubectl get pods --all-namespaces # or -A
```

To create a namespace with imperative commands: `kubectl create namespace my-new-ns`.

To create a namespace with yaml manifest:

```
kind: Namespace
apiVersion: v1
metadata:
  name: my-new-ns
```

To create a pod in a namespace with an yaml manifest, one sigle line must be added:

```
# nginx2-pod.yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx2
  namespace: my-new-ns
spec:
  containers:
    - name: nginx
      image: nginx
# eof
```

If you wish you can set a default namespace with:
```
kubectl config set-context --current --namespace=my-namespace
```

---

## Kubectl style commands

```
kubectl <VERB> <RESOURCE-TYPE> [<RESOURCE-NAME>]

kubectl get|describe|create|delete  nodes|pods|namespaces [-n namespace]
```

---

## Exercise with namespaces

- delete all the pods created in namespace default 
- create a new namespace, create a new nginx pod in this namespace
    - with imperative commands
    - with yaml manifests
- delete the new namespace created above. what's happening with the pods from namespace?

---

## Exercise with dummy pod

For demo-ing and sometimes for debugging, I have a handy container image which I like to call it `dummy`. From this I can create pod(s) and do stuff.

Your exercise here is to create a container image from the `Dockerfile`, then to start a pod from this image, then to enter (exec) into this pod and look around. Use the `dummy` directory in from this repo.

## More on PODs

As a final recap on pods:
- we have shown only a few pod parameters, there are many more and some of them will be uncovered later
- pods can have one or multiple containers, we'll have example of this later in this course
- the pod is the *atomic* entity in Kubernetes, a pod runs on a single node (Pod cannot "span" multiple nodes)
- The containers in a Pod can crash, they may or may not get restarted (depending on Pod's restart policy)



