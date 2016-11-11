# Docker
## 1. Install
Requires Kernel 3.0+, and update library before install

[Install on CentOS Guidance](https://docs.docker.com/engine/installation/linux/centos/)

## 2. Introduction
### 2.1 Usage
container, isolate applications
packaged with all its dependencies and libraries(environment)

### 2.2 Workflow
1. Get codes and dependencies tio container
2. Configure network or storage(optional)
3. Upload builds to a registry
4. Swarm cluster and scale(optional)
5. Deploy

A kind of _namespace_ tool?
Concentrates on 调度 and 编排 ？

### 2.3 Structure
#### 2.3.1 Components
1. Docker Client: UI, communicate with Daemon
2. Docker Daemon: sits on host, answers request
3. Docker Index: centralized registry

#### 2.3.2 Elements
1. Docker Containers: responsible for app's running, including OS, user files and meta data
2. Docker Images: read-only templates, help launch Docker containers
3. DOckerFile: file housing instructions, help automate image creation

[Docker Structure](http://blog.flux7.com/hs-fs/hub/411552/file-1222264954-png/blog-files/image-1.png?t=1476806838474&width=696&height=392&name=image-1.png)

#### 2.3.3 Support by OS
1. Namespaces: first level of isolation
2. Control Groups: a part of LXC(Linux Control), an OS level virtualization method for running multiple isolated Linux systems(containers)
3. UnionFS: file system

docker namespaces --> namespaces --> physical memory ?

#### 2.3.4 Docker Registry

Shelve progress:

[Docker入门教程（四）Docker Registry](http://dockone.io/article/104)

And official doc and guide:
[RA_CI with Docker](https://www.docker.com/sites/default/files/RA_CI%20with%20Docker_08.25.2015.pdf)

[Hello world in a container](https://docs.docker.com/engine/tutorials/dockerizing/)

## 3. Official getstarted
### 3.1 starter
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

### 3.2 Manage images
1. list images
> $ docker images

2. remove image
``` 
    $ docker rmi -f 7d9495d03763
    $ docker rmi -f docker-whale
```

3. daemonized container
```
query Docker daemon, list containers
[docker@dockermount docker]$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
657b964200ef        ubuntu              "/bin/sh -c 'while tr"   About a minute ago   Up About a minute                       suspicious_kilby
Or add flag -l, show only last container started.
add flag -a, show all containers.
add flag -q, show container ids only.

We can use NAME of this container to query execution logs:
[docker@dockermount docker]$ docker logs suspicious_kilby
It looks inside the container, and shows the standard output of a container.

And to stop this container:
[docker@dockermount docker]$ docker stop suspicious_kilby

```

### 3.3 Run container
1. Run command
> $ docker run ubuntu /bin/echo 'hello ubuntu'

2. Run interactive container
> $ docker run -t -i ubuntu /bin/bash
-t assigns a pseudo-tty or terminal inside the container
-i grabd teh standard input[STDIN] of the container, allow you to make an interactive connection

### 3.4 Run application
```
See port mapping:
$ docker port #name 5000

View logs:
$ docker logs -f #name

List processes:
$ docker top #name

Stop and remove
$ docker stop #name
$ docker rm #name

Remove all container:
$ docker rm $(docker ps -aq)
```
#### Note
**Removing a container is final(cannot undo)**

### 3.5 Network container
1. Name your container:
> $ docker run -d -P --name web training/webapp python app.py

2. Docker provides two network drivers: _bridge_(Default) and _overlay_.
Use _network_ sub command to list them:
> $ docker network ls

3. View Network info of container:
> [docker@dockermount ~]$ docker network inspect bridge

4. Create your own network:
> [docker@dockermount ~]$ docker network create -d bridge my-bridge-net

5. Connect your newly running container to this network:
> [docker@dockermount ~]$ docker network connect my-bridge-net web

6. Then try _ping_ each other in two containers:
> $ docker exec -it db bash

#### Note
**You can attach many network to a container, but two containers can reach other only in the same network**

### 3.6 Manage data in container
1. Data volume
A specially-designed directory, within one or more containers that bypasses the Union File System(operates by creating layers, making them very lightweight and fast)
Designed to persist data, even container is removed

Add a data volume to container:
> [docker@dockermount ~]$ docker run -d -P --name web -v /webapp training/webapp python app.py

**To be continue....**

### 3.7 Docker Certicates for both daemon and client
For detailed procedures:
[Protect the Docker daemon socker](https://docs.docker.com/engine/security/https/)

**Note:**
By default, Docker runs via a non-networked Unix socket. Config using an HTTP socket if you need communicate.

**$HOST in the doc is the ip addr of daemon server, not dns**

$ echo subjectAltName = IP:10.200.157.84,IP:10.200.157.48,IP:127.0.0.1 > extfile.cnf

**Mind**
To integrate with Spring boot on Windows, plz install docker-install.exe and DockerToolbox-1.12.2.exe. They will auto create a vm with installed docker.

Connect to docker VM by docker/tcuser

## Notice
### TO remember
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

### Benefits
1. Try new Tech at low cost
Using images in the Hub

2. Test and run with consistency

3. Build a dev environment 




## TO Be continue
[Manage data in container](https://docs.docker.com/engine/tutorials/dockervolumes/)
