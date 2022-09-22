# Dummy container

How to build and test it with docker:

```
docker build -t dummy:0.1 .
docker run -d  -p 6789:80 --name dummy dummy:0.1
docker ps
curl http://localhost:6789/cgi-bin/status
docker rm -f dummy
```


How to push it to a repository:

```
# example localhost:5000
docker tag dummy:0.1 localhost:5000/dummy:0.1
docker push localhost:5000/dummy:0.1

# example docker hub - requires account
docker login
docker tag dummy:0.1 vvang/dummy:0.1
docker push vvang/dummy:0.1
```

To run it, using `dummy-pod.yaml` file:

```
  kubectl apply -f dummy-pod.yaml 
  kubectl get pods
```
