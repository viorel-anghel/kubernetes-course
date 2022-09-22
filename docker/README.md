# Docker containers intro

## What are containers

Containers are 
  - a packaging mechanism for an executable unit of software 
  - application code is packaged with all the libraries and dependencies needed
  - in a  standardized way 
  - it can be run anywhere* (desktop, servers, cloud)
    - *where we have a 'docker engine'

Containers uses a form of virtualization based on Linux kernel features:
- namespaces - to isolate processes
- cgroups (control groups) - to limit the amount of CPU/memory/disk those processes have access to.

Year 2013 is considered to be the start of the modern container era - with the introduction of Docker.

Precursors: FreeBSD Jails - year 2000. https://en.wikipedia.org/wiki/FreeBSD_jail 

---

## Containers benefits
- isolation (CPU/memory/disk/network "sandboxed") - this is also achieved with VMs
- consistent environment - "it works on my computer" sindrome
- perfect fit for modern development patterns and architectures: microservices, Agile development
- portability - aka "run everywhere" (for example the same container can be run on different OS-es)
- lightweight (compared to VM) - small size, fast startup time

---

## What is Docker
- Docker - software that runs on Linux (and Windows) and can create and run containers
- Docker Inc - the company behind this
- OCI - Open Container Initiative - to make an open standard for containers (images and runtimes)

---

## Installing Docker

We'll use Rancher Desktop which is a free and opensource way to have Docker (and Kubernetes) or desktops/laptops with Windows, Linux or MacOS. 

Follow the info from this public repo: https://github.com/viorel-anghel/using-rancher-desktop . At the end of that
you should be able to use `docker` command.

For installing Docker on various Linux distributions, the official documentation is at 
https://docs.docker.com/engine/install/ 

---

## Working with Docker

### Docker commands intro

- docker command is the client part of the client-server Docker architecture
- `docker help` - this may be overwhelming
- important docker entities:
  - container - running processes
  - image - similar with a VM image but for containers
  - network - containers can be run in a virtual network and listen on network ports
  - volume - storage management
  - etc

### Short vs long docker commands

| short form    | long form (entity)    | explanation                            |
| ------------- | --------------------- | -------------------------------------- |
| docker ps     | docker container ls   | list running containers                |
| docker run    | docker container run  | start a new container                  |
| docker images | docker image ls       | list images in local cache             |
| docker pull   | docker image pull     | pull (download) an image from registry |


`docker ENTITY help`

---

### Starting and managing containers

```
docker run -d nginx      # -d = (daemon) run in background
docker ps
```

NOTE:
- automatic image pull 
- image layers
- images are cached
- container ID shown when ran
- discuss columns from docker ps

More info about containers

```
docker container inspect $ID
docker container logs $ID
docker container port $ID # list port mappings
docker container top $ID  # processes inside container
docker container stats # real time resource usage (like top) -> CTRL+C
```

IMPORTANT: if you start a container without `-d` (daemon / background), uou will not be able 
to stop it with `CTRL+C`, try `CTRL+\`.

---

### Starting with port mapping

```
docker container kill 91
docker run -p 8080:80 -d nginx
docker ps
docker container port 2c

nc -vz localhost 8080
curl http://localhost:8080
```

The argument -p is doing the port mapping. The left number is a port on the host, the right number a port inside container. 

```
docker run -p 8081:80 --name nginx2 -d nginx
docker run -p 8080:80 --name nginx1 -d nginx
```

---

### Stopping and deleting

`docker container`
- kill
- stop / start
- rm - delete a stopped container
- ps -a - show all containers, running and stopped
- pause / unpause

What is the difference between `docker stop` and `docker kill`? Both will stop the container, 
but 'stop' is graceful: send SIGTERM, and then SIGKILL after grace period.

---

### "Entering" in a container

```
docker container exec -ti 2c bash  # -ti IMPORTANT # 
    ps -ef
    ls -la
    cat /proc/1/cmdline
     exit
```

### Copying files in/from containers

```
docker container cp 2c:/etc/nginx/nginx.conf .
# SRC - DST 
```

---

### Important notes about containers
- startup time versus VM is better with at least one order of magnitude
- virtualization overhead (raw processing power loss) is almost nonexistent
- containers are based on two Linux kernel mechanism: cgroups (for resource limiting) and namespaces
- from the same image you can run multiple containers each with its own namespaces
- the container run a single process (usually) and when that process is terminated, the container will stop

---

### Exercise

The general syntax for starting containers is
```
docker run [flags] IMAGE [WHAT_to_RUN_INSIDE_CONTAINER]
```

Run and prove the last point using something like (explain commands):
```
docker container run -d alpine:latest sleep 30
docker container run -it ubuntu:latest bash
```

---

### Final exercises

- start a container with mysql. read the documentation here: https://hub.docker.com/_/mysql 
- set up a port mapping 13306 -> 3306
- test the port mapping (nc/telnet port 13306 )
- try
  - entering the container
  - listing mysql files, where are they?
  - list running processes
  - stop it and list all containers (both running and stopped)
  - stop all containers and cleanup everything (docker container rm)

---

#### HINT
mysql image requires a variable MYSQL_ROOT_PASSWORD :
```
docker run --name mysql1 -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql
                       # -e = environment
```

---

## Docker images

A Docker image is a unit of packaging that contains everything required for an application to run:
- application code (plus the interpreter if it's an interpreted language)
- application dependencies, shared libraries
- configuration files
- OS constructs. 

If you have an application’s Docker image, the only other thing you need to run that application is a computer running Docker engine.

### Docker image versus container
- Image = app we want to run plus all libraries, dependencies
- Container = instance of an image running as a process
- We can have many containers running from the same image
- Docker Hub = the default docker image registry (hub.docker.com)

### Image content
- Images are made up of multiple layers that are stacked on top of each other and represented as a single object.
- Inside of the image is a minimal operating system (OS) and all of the files and dependencies required to run an application. 
- Because containers are intended to be fast and lightweight, images tend to be small 

---

### Image registry

- You get Docker images by pulling them from an image registry. 
- The most common registry is Docker Hub (but others exist) https://hub.docker.com
- The image pull operation downloads the image to your local Docker host where is cached and then Docker can use it to start one or more containers.

```
docker image ls       # show locally cached images
docker image rm nginx
docker image pull nginx
docker info | grep Registry
```

- Image registry -> one or more (usually more) image repositories -> one or more versioned images
- Docker hub has official (for example nginx, mysql etc) repositories and unofficial repositories

--- 

### Pulling images from various sources

```
docker image pull nginx:latest
docker image pull vvang/dummy:0.3   # from https://hub.docker.com/u/vvang 
docker image pull gcr.io/google-containers/git-sync:v3.1.5
```

Free and open source image repo software, which can be installed on premises: Nexus, Jfrog, Harbour.

### Using a local image registry

Local image registry -- see https://docs.docker.com/registry/
```
docker image pull nginx:latest
docker image tag nginx:latest localhost:5000/nginx:2
docker image push localhost:5000/nginx:2
```

## Docker volumes

- Docker images are read-only objects (you can eventually create new images with the modified data) 
- Containers are designed to be immutable, they are good for stateless applications

- non-persistent data: we don't need to keep this outside container lifecycle
  - for example: logs, caching, one-time processing etc
  - each container will have its own non-persisting storage
  - stopping AND deleting the container will delete this data!
  - each container will have a *thin RW layer* for this data
- persistent data: 
  - we need to preserve this
  - containers can have one or more VOLUME(s) to store this data
  - volumes  are separate objects with lifecycle decoupled from containers
  - deleting a container with volume(s) will not delete the volume(s)

---

### Docker volume implementation

- Data volumes persist even if the container itself is deleted.
- Data volumes can be shared and reused among containers.
- Volumes are in fact directories on the host and they appear inside container as a mounted filesystem

---

### Named volumes vs mounts

You can have *named volumes* and *simple mounts*, the difference between them is: if it starts with **/** then it is a reference to a absolute path, otherwise is a named volume:

```
docker run -d --name nginx1 -v data:/data nginx 
docker volume ls
docker volume inspect data

docker run -d --name nginx2 -v /data:/data nginx

docker inspect nginx2  # check section Mounts
docker inspect nginx1  # check section Mounts
```

Another, longer syntax for volumes is `docker run ... --mount source=myvol,target=/data`.

### Exercise: sharing data between containers using volumes

```
docker run -d --name ubuntu1 --mount source=myvol,target=/data ubuntu sleep 1h

docker run -d --name ubuntu2 --mount source=myvol,target=/data ubuntu sleep 1h

docker exec -ti ubuntu1 bash
    touch /data/testfile
[...]
```

### Exercise: mysql data volume

Start a mysql container but ensure you will keep the data volume under /root/mysql. Verify it is ok after deleting the container.

path inside container: /var/lib/mysql

### Hints
```
apt install mysql-client
mkdir /root/mysql
docker run -d \
     --name mysql1 \
     -v /root/mysql:/var/lib/mysql \
     -p 1234:3306 \
     -e MYSQL_ROOT_PASSWORD=123456 \
     mysql

mysql -h 127.0.0.1 -P 1234 -p
    CREATE DATABASE abc;
    SHOW DATABASES;
```

### Cleaning up unused volumes

Named volumes can be deleted with `docker volume rm <volume-name>`. Directories used as mounts are not handled as docker objects.

```
docker volume prune

WARNING! This will remove all volumes not used by at least one container.
Are you sure you want to continue? [y/N] y
```

### Full docker cleanup

```
docker system prune --volumes
```





