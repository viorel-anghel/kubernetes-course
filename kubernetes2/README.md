# Kubernetes workloads

## Labels and annotations

- In a normal Kubernetes cluster we'll have hundreds or thousands of pods (and other resources). 
- The namespaces offer some organizatoric confort, but not enough. 
- In Kubernetes, the standard mechanism for selecting is to use *labels and selectors*.
- Labels (and annotations) are ways to attach metadata to objects in Kubernetes. 
- Labels are for both Kubernetes and humans and can be used as selectors
- Annotations a weaker mechanism, usually for humans, i.e. metadata Kubernetes does not care about.

Both labels and annotations are `key: value` with both fields definde by us. Example labels:
- "release" : "stable", "release" : "canary"
- "environment" : "dev", "environment" : "qa", "environment" : "production"
- "tier" : "frontend", "tier" : "backend", "tier" : "cache"
- "partition" : "customerA", "partition" : "customerB"

---

### Labels in manifest files

Example how to define label and annotations for a pod, in a manifest file:
```
# label definition demo
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
  annotations:
    owner: vang
spec:
  containers:
    - name: nginx
      image: nginx
```

---

### Labels in kubectl commands

```
kubectl get pods --show-labels
kubectl label pod nginx new-label=true # add a new label to a running pod
kubectl label pod nginx new-label-     # delete a label


# -l to limit by label
# -L to show a label as column
kubectl get pods -l run=nginx -L run

kubectl get nodes -L ingress-ready # show only the nodes with this label
```

---

## Replicaset

Pods are rarely used in production clusters, because there are some higher level objects, based on pods, which offer various advantages. We'll discuss some of them.

- A *ReplicaSet* is a set of identical (replicated) Pods
- Defined by a pod template + number of desired replicas
- If there are not enough Pods, the Replica Set creates more (e.g. in case of node outage; or simply when scaling up)
- If there are too many Pods, the Replica Set deletes some  (e.g. if a node was disconnected and comes back; or when scaling down)
- We can scale up/down a Replica Set
  - we update the manifest of the Replica Set
  - as a consequence, the Replica Set controller creates/deletes Pods

In practice, you may never need to manipulate ReplicaSet objects: the next workload object is better. 

---

## Deployment


