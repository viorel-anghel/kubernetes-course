# Building docker containers and using image repositories

## Containerizing an application

The whole containers idea is to make simple for an application to be deployed. Docker mantra is *build, ship, run*:

- build
  - start with application code and dependencies
  - create a file called Dockerfile which describe this
  - use the 'docker build' to create a docker image
- ship
  - push the new image into a registry (optional)
- run
  - start container(s) from that image

## Exercise: mysql image with embedded password

```
### Dockerfile
FROM mysql
ENV MYSQL_ROOT_PASSWORD=123456
```

```
# run command
docker build -t mysqlpw .
```

## Dockerfile useful images FROM

When you create a new image, it's useful to start FROM a specific image. Search https://hub.docker.com/ for good ones:

- classic Linux distributions: Ubuntu, Debian, Centos
  - big images but fully featured
- busybox
  - very small image, basic utilities, NO package manager
- alpine
  - small image, minimal Linux with package manager (apk)
- scratch ! - Ubuntu - FROM scratch

- dedicated to various programming languages:
  - openjdk
  - node
  - php
  - etc

- dedicated to specific applications:
  - nginx
  - mysql
  - postgresql
  - redis
  - etc

---

## Exercise: containerising a simple app ("build it")

```
# this is myapp.sh
while :
do
  echo $RANDOM | nc -q 1 -l 8888
done
# eof
```

```
# this is Dockerfile
FROM ubuntu:20.04
RUN apt-get update -y
RUN apt-get install -y netcat
COPY myapp.sh /root/myapp.sh
WORKDIR /root
EXPOSE 8888
ENTRYPOINT ["/bin/bash", "./myapp.sh"]
# eof
```

Create a new directory with those two files ('mkdir myapp') and inside that directory ('cd myapp') run

```
docker build -t myapp:latest .    # don't forget the last dot!!!

docker image ls
docker run -d -p 18888:8888 myapp:latest
docker ps

nc 0 18888
```

---

## Pushing the image to a repository ("ship it")

example build with multiple tags:
```
docker build -t myapp:1.0.2 -t myapp:latest .
```

use docker push to upload an image to a registry, but first we must tag-it with a special syntax:

```
docker login # login to docker hub
docker image tag myapp:latest [registry-url]/repository/myapp:latest
docker image ls
docker image push [registry-url]/repository/myapp:latest
```

---

## Running the app ("run it")
We already saw that "docker run" is pulling images from the registry when needed.

---

# Volumes in dockerfile

Example from the official mysql image:
https://github.com/docker-library/mysql/blob/3362baccb4352bcf0022014f67c1ec7e6808b8c5/8.0/Dockerfile

```
    [...]
VOLUME /var/lib/mysql
    [...]
```

Every time you create a container from this image, docker will force that directory to be a volume.

You cannot specify a volume source in the Dockerfile. The docker run command will specify the volume source with --mount or -v:

```
docker run -v /my/own/datadir:/var/lib/mysql mysql
```

This command will mount the `/my/own/datadir` directory from the underlying host system as `/var/lib/mysql` inside the container.

---

## User security in Dockerfiles

- For security considerations, it is highly recommended to run your applications as a non-root user even inside containers
- But by default, all actions in Dockerfiles are ran as root

A common pattern to solve this:

```
# example Dockerfile
From ubuntu
# runs as root
RUN apt-get update -y && apt-get upgrade -y

# switch to a non-root user
RUN useradd --create-home appuser
WORKDIR /home/appuser
USER appuser

# runs as non-root user
ENTRYPOINT ["whoami"]
```

```
docker build -t myapp2:latest .
docker run myapp2
```

### Exercise
Rebuild myapp Exercise: containerising a simple app ("build it") images/Dockerfile to use a non-root user for running the application inside the container as explained above.

---

## Image entrypoint
- The important (but optional) parameter *ENTRYPOINT* in Dockerfile
- Docker run will execute the ENTRYPOINT command inside the container and will keep the container alive as long as this process is running

Exec versus shell syntax:
```
ENTRYPOINT ["executable", "param1", … ]  ### runs as PID=1

ENTRYPOINT command param1 ...            ### runs under /bin/sh
```

---

### Some common entry-points

- example 1 - run a daemon in foreground
```
FROM ubuntu
RUN apt-get update && apt-get install -y nginx
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

- example 2 - a shell script
```
COPY ./docker-entrypoint.sh /
ENTRYPOINT [ "/docker-entrypoint.sh" ]
```


- example 3 - forever sleep but with "logs"
```
From ubuntu
ENTRYPOINT while : ; do echo . ; sleep 60; done
```

- example 4 - `ping 127.0.0.1`

---

## Entrypoint versus CMD

- CMD is somehow similar with ENTRYPOINT
- do not confuse CMD with RUN!
- only the last CMD in a Dockerfile is used
- this can be confusing and I usually avoid it but here is an example where CMD is useful

```
# instead of
CMD ["sleep","5"]

# or
ENTRYPOINT ["sleep"]

# is better to use
FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["10"]

# the resulting container can be run with or without parameter:
docker run -d ubuntu-sleeper
docker run -d ubuntu-sleeper 30
```

### Exercise

- inherit the nginx image to create a new docker image named mynginx
- which shall respond to http requests with text "OK"
- run and verify
- enter "inside" container and show the file index.html

HINT: alter the file `/usr/share/nginx/html/index.html`

---

## Using docker as a build platform

Docker images can be used as a build or test platform, especially the standard distribution images where you can choose a specific distribution and version.

```
/* file hello.c */

#include<stdio.h>

int main(void) {
    printf("Hello World\n");
    return 0;
}
```

```
apt install gcc
gcc hello.c
./a.out
```


### EXERCISE: containerize this app.

```
# Dockerfile
FROM ubuntu
COPY a.out /root/a.out
ENTRYPOINT [ "/root/a.out" ]
```

---

## Docker multi-stage build

- to keep the docker images small, without the build tools
- to keep the source cod private
- to use a consistent build environment

Observe the strange thing, two FROM instructions:

```
# Dockerfile
FROM ubuntu:18.04
RUN apt-get update -y
RUN apt-get install -y gcc

WORKDIR /tmp
COPY hello.c .
RUN gcc hello.c

FROM ubuntu:20.04
WORKDIR /root
COPY --from=0 /tmp/a.out .
ENTRYPOINT [ "./a.out" ]
```

```
docker build -t aout:latest .
docker run aout
```

---

### Using named stages

```
# Dockerfile
FROM ubuntu:18.04 as builder
RUN apt-get update -y
RUN apt-get install -y gcc

WORKDIR /tmp
COPY hello.c .
RUN gcc hello.c

FROM ubuntu:20.04
WORKDIR /root
COPY --from=builder /tmp/a.out .
ENTRYPOINT [ "./a.out" ]
```

```
docker run aout
```

---

## Docker image debugging

When building images

```
docker build -t sleeper  .
Sending build context to Docker daemon  2.048kB
Step 1/3 : From ubuntu:18.04
 ---> 3339fde08fc3
Step 2/3 : WORKDIR /tmp
 ---> Using cache
 ---> 949ee6a4ee5e
Step 3/3 : ENTRYPOINT while : ; do echo . ; sleep 60; done
 ---> Using cache
 ---> a9f7c42c869b
Successfully built a9f7c42c869b
Successfully tagged sleeper:latest
```

At every line from Dockerfile, a new image is created. You may use and run those images for debugging:

```
docker run --name debug1 -it 3339fde08fc3 bash
```

or use 'docker image inspect' for image details.

---

## Debugging running containers

- use `docker logs` to check container STDOUT/STDERR
- use `docker exec` to enter a running container:
`docker exec -ti b1 bash`
- use `docker top` to see the running processes
- use `docker inspect` to get container details


---

## Dockerfile reference

Official documentation: https://docs.docker.com/engine/reference/builder/





