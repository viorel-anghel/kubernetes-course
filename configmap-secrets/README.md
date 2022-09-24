# ConfigMaps and Secrets

## Environment variables

To avoid hardcoding variables in container images, in Kubernetes, like in Docker, you can use environment variables. 

To test this, apply this yaml file, then:

```
# env-example.yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-example
spec:
  containers:
    - name: alping
      image: alpine:3.11
      command: [ "ping", "127.0.0.1" ]
      env:
        - name: VAR1
          value: "hello"
        - name: VAR2
          value: "World!"
```

```bash
less env-example.yaml 
kubectl apply -f env-example.yaml 
kubectl get pods
kubectl exec -ti env-example -- sh
    # echo $VAR1 $VAR2
    hello World!
    # exit
kubectl delete pod env-example
```

---

## Config maps

While this is nice and almost the same as for docker, Kubernetes has a special and dedicated object for such configurations and environment variables, called `ConfigMap`. 

This is useful to avoid repetition when the same environment variables are used by many objects.

A configmap is a Kubernetes resource that exists in a namespace, conceptually, it's a key/value map (values are arbitrary strings).

Note: to hold sensitive information, we can use "Secrets", which are another type of resource behaving very much like configmaps.

### ConfigMap example
Please note this is a different object and it has it's on lifecycle:

```
# cm-example.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-example
data:
  varA: "hello"
  varB: "World!"
---
apiVersion: v1
kind: Pod
metadata:
  name: cm-example
spec:
  containers:
    - name: alping
      image: alpine:3.11
      command: [ "ping", "127.0.0.1" ]
      env:
        - name: VAR1
          valueFrom: 
            configMapKeyRef:
              name: cm-example
              key: varA
        - name: VAR2
          valueFrom: 
            configMapKeyRef:
              name: cm-example
              key: varB
```

```bash
less cm-example.yaml
kubectl apply -f cm-example.yaml

kubectl exec -ti cm-example -- sh
    # echo $VAR1 $VAR2
    hello World!
    # exit

kubectl delete configmap/cm-example pod/cm-example 
```

### ConfigMaps as mounts

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-demo
data:
  # simple values
  value1: "1"
  value2: "2"
  # multi line config file
  config_file: |
    some lines
    in a demo
    config file

---
kind: Pod
apiVersion: v1
metadata:
  name: dummy-cm
spec:
  volumes:
    - name: config
      configMap:
        name: cm-demo
  containers:
    - name: dummy
      image: vvang/dummy:0.3
      volumeMounts: 
        - name: config
          mountPath: /config
          readOnly: true
```

```
kubectl apply -f 40-configmap.yaml
kubectl get cm cm-demo -o yaml

kubectl exec -ti dummy-cm  -- bash
    ls -la /config
```

---

## Secrets

- Some information must be kept secure. For this Kubernetes provides a separate object called a Secret. 
- Secrets are much like ConfigMaps—they’re also maps that hold key-value pairs. 
- They can be used the same way as a ConfigMap:
  - You can pass Secret entries to the container as environment variables
  - or expose Secret entries as files in a volume
- Kubernetes helps keep your Secrets safe 
  - by making sure each Secret is only distributed to the nodes that run the pods that need access to the Secret. 
  - Also, on the nodes themselves, Secrets are always stored in memory and never written to physical storage

### Secret examples
Here are a few commands to play with secrets (secrets are namespaced, just like configmaps):

```bash
kubectl create ns secret-test
kubectl -n secret-test create secret generic my-secret --from-literal=key1=supersecret
kubectl -n secret-test get secret  my-secret -o yaml
```

```
# secret-example.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: secret-example
  namespace: secret-test
spec:
  containers:
    - name: alping
      image: alpine:3.11
      command: [ "ping", "127.0.0.1" ]
      env:
        - name: VAR1
          valueFrom: 
            secretKeyRef:
              name: my-secret
              key: key1
```

Now you can run and verify this pod:
```bash
kubectl apply -f secret-example.yaml 
kubectl -n secret-test exec -ti secret-example  -- sh
    # echo $VAR1
    # exit
```
