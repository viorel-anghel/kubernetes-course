# Kubernetes storage

## Kubernetes storage

As with Docker, Kubernetes is very good when using stateless containers. But nowadays, the trend is to 
run everything in Kubernetes, including databases.

The concept of volume is used to provide persistence. In Kubernetes there are many types of volumes, but only a few types are relevant form the developer point of view:

    emptyDir - used for for sharing files between containers in the same pod. will start empty again when pod restart
    hostPath - a file or directory on the node filesystem
    configMap, secret - special cases, those can be mounted as virtual filesystems
    persistentVolumeClaim (PVC) - the really-really persistent volumes in Kubernetes

## emptyDir volumes

- emptyDir volumes are defined as a part of a pod and share the same lifecycle as the pod
- the volume is created when the pod is started and is destroyed when the pod is deleted
- best use case for this:
  - you can share directories between containers in the same pod
    - a pod can have multiple containers
    - Containers in a POD will share resources such as CPU, RAM, network interfaces
    - but not the "DISK", each container will have its own isolated filesystem (since the filesystem comes from the container’s image)
 
---

### Example pod with two containers and a shared volume

*emptyDir* volume type is simple empty directory used for storing transient data. It will not persist data across pod restarts.

```
# empytdir is used in pods with multiple containers

apiVersion: v1
kind: Pod
metadata:
  name: example-emptydir
spec:

  volumes:
  - name: vol1
    emptyDir: {}

  containers:
  - name: c1
    image: alpine
    command: [ "ping", "127.0.0.1" ]
    volumeMounts:
    - name: vol1
      mountPath: /data
  - name: c2
    image: nginx
    volumeMounts:
    - name: vol1
      mountPath: /data-nginx
```

---

### Multi container pods - why?

Pods will often only have one container - this is normal! When should you combine multiple containers into a single pod?

The most common scenario is that you have a helper process that needs to be located and managed on the same node as the primary container. Another reason to combine containers into a single pod is for simpler communication between containers in the pod. 

Those containers can communicate 
- through shared volumes (writing to a shared directory) 
- through inter-process communication (semaphores or shared memory).
- or via networking on localhost

Design patterns:
- *sidecar pattern* - POD consists of a main application i.e. your web application plus a helper container with a responsibility that is essential to your application, but is not necessarily part of the application itself. The most common sidecar containers are logging utilities, sync services, watchers, and monitoring agents.
- *adapter pattern* - is used to standardize and normalize application output or monitoring data for aggregation.
- *ambassador pattern* is a useful way to connect containers with the outside world. An ambassador container is essentially a proxy that allows other containers to connect to a port on localhost while the ambassador container can proxy these connections to different environments depending on the cluster's needs.

---

### hostPath volumes

Hostpath is simple and nice but cannot be really used for persistence in clusters with multiple nodes because you usually don't control on which node your pod will be started.

In this example we show how you can mount the *full node filesystem* and see it inside the container under /mnt. 

```
# hostpath is used to access the NODE filesystem

apiVersion: v1
kind: Pod
metadata:
  name: example-hostpath
spec:

  volumes:
  - name: vol1
    hostPath: 
      path: /

  containers:
  - name: c1
    image: alpine
    command: [ "ping", "127.0.0.1" ]
    volumeMounts:
    - name: vol1
      mountPath: /mnt
```

```
kubectl get pods

kubectl exec -ti example-hostpath -- sh
    ls -l /mnt
    exit

kubectl delete -f example-hostpath.yaml
```

---

### SC and PVC

Let's discuss now about the real persistence in Kubernetes.

- the simple storage volumes are destroyed when the pod is deleted! 
- Networked storage like NFS or gcePersistentDisk are ok, but they have the disadvantage that you must be aware of the infrastructure, which is against Kubernetes principles
- two new resources were introduced: PersistentVolumes (PV) and PersistentVolumeClaims (PVC)

- PV is a storage block within the cluster and therefore is a cluster level resource. 
  - The PV can be either provisioned to the user by the k8 administrator 
  - or can dynamically be provisioned if Storage Classes are available in k8.
  - hose PV can be consumed by Pods by issuing a PersistentVolumeClaim object -- a PVC. 
- A PVC object lets pods use storage from Persistent Volumes. 
  - PVC are namespaced
- A storageclass is a Kubernetes object that stores information about creating a PV. 
  - With a storageclass, you do not need to create a persistent volume separately before claiming it.

For developers, two objects are important: storageClass and persistentVolumeClaim. Once you have access to a new Kubernetes cluster, first thing you should do regarding to persistence is to run
```
kubectl get storageclasses # sc
```

In some environments, it's possible to have more than one storage class and then you should ask / decide which one to use.

### Example pod and PVC

With the storage class in place, you will only need to define a Persistent Volume Claim - observe this is a standalone object, outside the pod - and a POD using it, as in the next example . This will automatically create a Persistent Volume:

```
apiVersion: v1
kind: Pod
metadata:
  name: example-pod-pvc
spec:
  containers:
  - name: vol1
    image: alpine
    command: [ "ping", "127.0.0.1" ]
    volumeMounts:
    - name: vol1
      mountPath: /data
  volumes:
  - name: vol1
    persistentVolumeClaim:
      claimName: pvc1

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

```
kubectl apply -f example1-pvc-pod.yaml 
kubectl exec -ti example-pod-pvc -- sh
    # touch /data/abc
    # exit

kubectl delete pod example-pod-pvc
kubectl get pvc
kubectl get pv
kubectl apply -f example1-pvc-pod.yaml 
```

You should note various storage classes have various limitations. For example, the one used by KIND does not support ReadWriteMany (RWM) access mode. But you can successfully use RWO with our single node setup!

Another important thing regarding storage and deployments is that there is a special kind called StatefulSet which is basically like a deployment plus persistent volumes.

### Deployments vs StafefulSet

- Deployment is a resource to deploy a stateless application, if using a PVC, all replicas will be using the same Volume and none of it will have its own state.
- Statefulsets is used for Stateful applications, each replica of the pod will have its own state, and will be using its own Volume.

https://stackoverflow.com/questions/41583672/kubernetes-deployments-vs-statefulsets

https://medium.com/stakater/k8s-deployments-vs-statefulsets-vs-daemonsets-60582f0c62d4


