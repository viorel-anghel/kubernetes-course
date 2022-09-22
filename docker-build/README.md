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

