# Kubernetes networking, services, ingress


## Kubernetes network model

- Kubernetes uses a virtual internal network (overlay over nodes IPs)
- each POD has it's own IP in this network
- pods can communicate over this big flat IP network, regardless on which node are they running
- there are other networking objects in this network
  - services act like loadbalancers
  - ingresses are like layar 7 loadbalancers, used to route external traffic to internal network
  - networkPolicies are like firewalls

## Exposing pods
- Kubernetes has a resource type named *Service* used to solve those problems:
  - pods get dynamic IP addreses, similar with DHCP
  - how do we connect from outside the cluster?
  - how do we load balance traffic?
  - what if a pod fails?

## Services
Services give us a stable endpoint to connect to a pod or a group of pods

An easy way to create a service is to use `kubectl expose`. If we have a deployment named my-little-deploy, we can run:
```
kubectl expose deployment my-little-deploy --port=80
```
and this will create a service with the same name (my-little-deploy).

## Advantages of services
- services have static IPs but also DNS names, no need to use the IP address of the pod(s)
- There are multiple service types; some of them allow external traffic (e.g. LoadBalancer and NodePort)
- Services provide load balancing (for both internal and external traffic), i.e they can send requests to multiple pods
- Service addresses are independent from pods' addresses (when a pod fails, the service seamlessly sends traffic to active pods

## Service ClusterIP
- It's the default service type
- A virtual IP address is allocated for the service (in an internal, private range; e.g. 10.96.0.0/12)
- This IP address is reachable only from within the cluster (nodes and pods)
- Our code can connect to the service using the original port number
- Perfect for internal communication, within the cluster

## Service NodePort
- A port number is allocated for the service (by default, in the 30000-32767 range)
- That port is made available on *all cluster nodes* and anybody can connect to it (we can connect to any node on that port to reach the service)
- Our code needs to be changed to connect to that new port number
- Under the hood: kube-proxy sets up a bunch of iptables rules on our nodes
- Sometimes, it's the only available option for external traffic

## Service Loadbalancer

An external (i.e. outside Kubernetes cluster) load balancer is allocated for the service - typically a cloud load balancer, e.g. ELB on AWS, GLB on GCE.

Fow on-premise clusters, there are solutions like *MetalLB* or *kube-vip* but some extra-setup is required.

## ReadinessProbe

In the container definition (in a pod or deployment or other workload object) we can also define a *ReadynessProbe*, similar with the LivenessProbe.

This is used by services to detect when a pod is ready to serve client requests. If the pod is not ready, the service will not route requests to it.

Probes are like in Liveness: HTTP GET, TCP Socket, Exec. For example, Your container code can answer to http requests for paths like:
- /live -- ok
- /ready -- ok

## Creating services

With imperative commands, you can use `kubectl expose ...`. 

For yaml manifests, this is an example:
```
# service example
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: myapp
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: myapp
```

```
kubectl get service myapp -n myapp
kubectl describe service myapp -n myapp
```

### Exercise with service
- create a deployment and a service to send requests to it  
- check accessing the service, using DNS name, from the same namespace and from another namespace, using a
testing/dummy pod

---

## Accessing Kubernetes from outside
- we want to access kubernetes services from OUTSIDE the cluster, not from inside
- one solution is the LoadBalancer service but we don't always have that luxury
- or with NodePort – high random port
- another solution is to use "kubectl port-forward" but this is for devops, not for regular users
- another solution is Kubernetes Ingress

Exercise
```
kubectl port-forward service/service-bar 12345:80
curl localhost:12345
```
---

### Example NodePort

```
# service example
apiVersion: v1
kind: Service
metadata:
  name: myapp-nodeport
  namespace: myapp
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: myapp
```

```
kubectl get svc
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
myapp-nodeport   NodePort    10.43.245.79   <none>        80:31646/TCP   5s
```


## Kubernetes nginx ingress
- Kubernetes Ingresses allow you to flexibly route traffic from outside your Kubernetes cluster to Services inside your cluster.
- This is accomplished using 
  - Ingress Resources, which define rules for routing HTTP and HTTPS traffic to Kubernetes Services, 
  - and Ingress Controllers, which implement the rules by load balancing traffic and routing it to the appropriate backend Services

### Ingress controller
- Ingress is a default resource in Kubernetes and define traffic rules
- Ingress Controllers are NOT efault in Kubernetes, there are many implementations
- some extra install steps are required, unless the Kubernetes distribution has already did that
- popular Ingress Controllers: nginx-ingress, traefik and others

### Ingress exercise

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp
  namespace: myapp
#  annotations:
#    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: "myapp.localdomain"
    http:
      paths:
        - pathType: Prefix
          path: "/"
          backend:
            service:
              name: myapp
              port:
                number: 80
```

```
curl --header 'Host: myapp.localdomain' http://<IP>/
```














