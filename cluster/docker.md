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