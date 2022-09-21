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

| docker ps     | docker container ls   | list running containers                |
| docker run    | docker container run  | start a new container                  |
| docker images | docker image ls       | list images in local cache             |
| docker pull   | docker image pull     | pull (download) an image from registry |


`docker ENTITY help`


