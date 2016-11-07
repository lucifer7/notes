# Docker
## Install
Requires Kernel 3.0+, and update library before install

[Install on CentOS Guidance](https://docs.docker.com/engine/installation/linux/centos/)

## Introduction
### Usage
container, isolate applications
packaged with all its dependencies and libraries(environment)

### Workflow
1. Get codes and dependencies tio container
2. Configure network or storage(optional)
3. Upload builds to a registry
4. Swarm cluster and scale(optional)
5. Deploy

### Structure
#### Components
1. Docker Client: UI, communicate with Daemon
2. Docker Daemon: sits on host, answers request
3. Docker Index: centralized registry

#### Elements
1. Docker Containers: responsible for app's running, including OS, user files and meta data
2. Docker Images: read-only templates, help launch Docker containers
3. DOckerFile: file housing instructions, help automate image creation

[Docker Structure](http://blog.flux7.com/hs-fs/hub/411552/file-1222264954-png/blog-files/image-1.png?t=1476806838474&width=696&height=392&name=image-1.png)

#### Support by OS
1. Namespaces: first level of isolation
2. Control Groups: a part of LXC(Linux Control), an OS level virtualization method for running multiple isolated Linux systems(containers)
3. UnionFS: file system

docker namespaces --> namespaces --> physical memory ?

#### Docker Registry

Shelve progress:

[Docker入门教程（四）Docker Registry](http://dockone.io/article/104)

And official doc and guide:
[RA_CI with Docker](https://www.docker.com/sites/default/files/RA_CI%20with%20Docker_08.25.2015.pdf)

[Hello world in a container](https://docs.docker.com/engine/tutorials/dockerizing/)

## Official getstarted
### starter
1. run a hello world to check
$ sudo docker run hello-word
run: create & run a container
hello-world: image to load into the container
? option -rm means ..?

2. list all containers
$ docker ps -a

image: is a filesystem and parameters to use at runtime
doesn't have state and never changes

Container: a running instance of an image

3. Find image in Docker Hub
[Docker Hub](https://hub.docker.com/explore/)

4. Build your own image
```
$ touch Dockerfile
Contents:
FROM docker/whalesay:latest
RUN apt-get -y update && apt-get install -y fortunes
CMD /usr/games/fortune -a | cowsay

Then build an image from Dockerfile:
$ docker build -t docker-whale .

The procedure:
1. Docker check things available: send context to Docker daemon
2. load image defined in FROM
3. RUN
4. CMD
```

5. Push image to Hub/repository
sign up for Docker Hub
create a repository for image
push image to online

> $ docker tag 292ad9b8d884 skyvoice/docker-whale:latest
> $ docker login
> $ docker push skyvoice/docker-whale

### Manage images
1. remove image
``` 
    $ docker rmi -f 7d9495d03763
    $ docker rmi -f docker-whale
```

## Notice
### TO remember/
#### Advantages
1. containers are immutable
same image tested by QA will reach production environment with same behaviour

2. container are lightweight
memory footprint of it is small

3. container are fast
DO NOT treat containers as virtual machines

#### Mentra
1. containers are disposable/ephemeral

#### Avoid
1. dont store data
2. dont ship application in two pieces
3. dont crate large images
dont install unnecessary packages or run "update" 
4. dont use a single layer image
use layered filesystem, username definition, runtime installation, configuration and then application
easier to recreate, manage and distribute
5. dont create images from running containers
6. dont use only "lastest" tag
like "SNAPSHOT", unsafe and irretrievable
7. dont run more than on process in a single container
8. dont store credentials in image
use environment variables
9. dont run processes as root
10. dont rely on IP addresses
use environment variables

**Source:**
[10 things to avoid in docker containers](http://developers.redhat.com/blog/2016/02/24/10-things-to-avoid-in-docker-containers/)