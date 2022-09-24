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

- Replica Sets control identical Pods
- Deployments are used to roll out different Pods
  - different image (version), command, environment variables, …
- When we update a Deployment with a new Pod definition:
  - a new Replica Set is created with the new Pod definition
    - that new Replica Set is progressively scaled up
    - meanwhile, the old Replica Set is scaled down
  - This is a rolling update, minimizing application downtime
- When we scale up/down a Deployment, it scales up/down its Replica Set

---

### Exercise with deployment
```
kubectl create deployment pingpong --image=alpine -- ping 127.0.0.1
kubectl get deploy
kubectl get rs
kubectl get pods
kubectl logs deploy/pingpong

# scaling
kubectl scale deploy/pingpong --replicas 2
kubectl get pods

# what happens if you manually delete a pod?
kubectl delete pod pingpong-xxxxxxxxxx-yyyyy
kubectl delete deployment pingpong
```

```
# what happens if one NODE goes down?
kubectl get pods -o wide
docker stop kind-worker
kubectl get nodes

# it will take some 10 minutes for kubernetes to start new pods 
# on the other worker, because there is no LivenessProbe
# but after some time...
kubectl get pods -o wide
kubectl get deploy

docker start kind-worker
kubectl get nodes
```

```
# restarting pods!
kubectl rollout restart deployment myapp
```

Important: Always use deployments, even for single pods! 
restartPolicy Always is not enough.

### Deployment using YAML file

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

```
### deploy1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alping-deployment
  labels:
    app: alping
spec:
  replicas: 2
  selector:
    matchLabels:
      app: alping
  template:
    metadata:
      labels:
        app: alping
    spec:
      containers:
        - name: alping
          image: alpine:3.11
          command: [ "ping", "127.0.0.1" ]
#eof

kubectl apply -f deploy1.yaml
kubectl get deploy
kubectl rollout status deployment/alping-deployment
kubectl get rs
kubectl get pods --show-labels
kubectl describe deployment alping-deployment
```

### Changing the image version
edit the deployment yaml - change image version to 3.12
```
kubectl replace -f deploy1.yaml
kubectl get rs

TBD: Q why kubectl replace and not apply
```

### Exercise
- create a deployment WITH YAML file for the dummy pod with 1 replicas
- increase the number of replicas to 2
- on which nodes are the pods running now?
- what is happening if you delete one pod with kubectl delete pod …?

---

### Health probes
- Kubernetes will restart automagically containers/PODs which are crashing
- But sometimes an app may be unresponsive without container crashing
- for this cases new checks can be defined:
- kubernetes can check if a container is alive via *liveness probe*. that can be
  - a HTTP GET probe
  - a TCP Socket probe
  - an Exec probe (arbitrary command ran inside container)

### Example YAML syntax for livenessProbe:

```
apiVersion: v1
kind: Pod
metadata:
 name: rng-with-liveness
spec:
 containers:
 - name: rng
   image: dockercoins/rng:v0.1
   livenessProbe:
     httpGet:
       path: /
       port: 80
     initialDelaySeconds: 10
     periodSeconds: 3
```

---

## Other workload objects
There are many other workload API objects but those are left for individual study. For example: 
- StatefulSet - similar with Deployment but for stateful applications (usually with storage volumes)
- DaemonSet - ensures that all (or some) Nodes run a copy of a Pod
- Job - run one or more pods until they successfully complete
- CronJob - creates Jobs on a repeating schedule.

https://kubernetes.io/docs/concepts/workloads/controllers/ 

---

## POD limits

- since those workload objects can create many PODs, it's wise to set some memory and CPU limits on containers
- for every container you can define 
  - CPU/memory requests (min resources) and 
  - CPU/memory limits (max resources)

Snippet - Example YAML syntax for resources (per container!) containers:

```
  - name: httpenv
    image: jpetazzo/httpenv
    resources:
      limits:
        memory: "200Mi"
        cpu: "100m" 
      requests:
        memory: "100Mi"
        cpu: "10m"
```

### POD QOS (quality of service)

- If limits = requests
  - as long as the container uses less than the limit, it won't be affected
  - if all containers in a pod have (limits=requests), QoS is called *Guaranteed*
- If requests < limits:
  - as long as the container uses less than the request, it won't be affected
  - otherwise, it might be killed/evicted if the node gets overloaded
  - if at least one container has (requests<limits), QoS is called *Burstable*
- If a pod doesn't have any request nor limit, QoS is considered *BestEffort*


WHEN A NODE IS OVERLOADED
- BestEffort pods are killed first
- Then, Burstable pods that exceed their requests
- Burstable and Guaranteed pods below their requests are never killed (except if their node fails)
- If we only use Guaranteed pods, no pod should ever be killed (as long as they stay within their limits)

For limits per namespace, see *ResourceQuotas*.








