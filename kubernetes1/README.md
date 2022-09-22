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

I'll share with you the kubeconfig file.

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


