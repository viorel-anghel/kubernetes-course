# Using docker-compose

Docker-compose: single host orchestration.

Why?
- microservices: multiple smaller services that interact to form a useful app
- deploying and managing lots of small microservices like these can be hard

Docker-compose

- Instead of using "docker run" commands and/or scripts
- with docker-compose you can describe and entire multi-container app in a single configuration file
- once the app is deployed, you can manage its life-cycle with simple commands

Docker-compose is a tool used to build, ship and run multi-container applications on a single docker host.

---

## Docker-compose features

- Multiple isolated environments on a single host
  - Uses a project name to isolate environments from each other
  - The default project name is the basename of the project directory.
  - You can set a custom project name by using the -p
- Preserve volume data when containers are created
- Only recreate containers that have changed
- Variables to customize the result
  - Compose uses the variable values from the shell environment
  - https://docs.docker.com/compose/compose-file/compose-file-v3/#variable-substitution 

---

## Installing docker-compose

https://docs.docker.com/compose/install/

```
docker-compose --version
```

---

## Sample docker-compose file and networking

```
# Dockerfile
FROM ubuntu
RUN apt-get update -y
RUN apt-get install -y iputils-ping
ENTRYPOINT while : ; do echo . ; sleep 60; done
```

```
# docker-compose.yml
version: "2"
services:
  ubuntu1:
    image: ubuntu-ping
  ubuntu2:
    image: ubuntu-ping
```

Commands:
```
docker build -t ubuntu-ping .
docker-compose up -d
docker ps
docker network ls
docker-compose down
```

- A new virtual network is created.
- A container is created for ubuntu1. It joins the network with the hostname ubuntu1.
- A container is created for ubuntu2. It joins the network with the hostname ubuntu2.
- Those containers can "see" each other in the same network with those names.

---

## Docker-compose building images
```
# compose.yml
version: "2"
services:
  ubuntu1:
    build:
        context: .
        dockerfile: Dockerfile
    image: ubuntu-ping:2.0
  ubuntu2:
     image: ubuntu-ping:2.0
```

---

## EXAMPLE - Wordpress with docker-compose

- Wordpress is a php application with web interface and mysql database. 
- Fortunately, we have  default images for wordpress and mysql.
- https://hub.docker.com/_/wordpress
- https://github.com/docker-library/wordpress/blob/dd8724a5b8c21eebd262aebf3593df0e13b5e917/latest/php7.4/apache/Dockerfile

```
# docker-compose wordpress.yml file
version: "2"
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
#end
```

Usage:
```
docker-compose -f wordpress.yml up -d # starting
docker volume ls
docker ps
curl http://localhost:8000
docker-compose -f wordpress.yml top
# stop here
docker-compose -f wordpress.yml down
docker-compose -f wordpress.yml down --volumes
```

---

## YAML syntax basics
https://www.tutorialspoint.com/yaml/yaml_basics.htm

- Indentation of whitespace is used to denote structure.
- Comments in YAML begins with the (#) character.
- List members are denoted by a leading hyphen (-)
- Associative arrays are represented using colon ( : ) in the format of key value pair
- use --- to separe multiple documents 



Documentation on docker-compose https://docs.docker.com/compose/


