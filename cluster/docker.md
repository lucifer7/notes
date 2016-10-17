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
1. Docker Containers: in charge of app's running, including OS, user files and meta data
2. Docker Images: a read-only template, to run Docker Container
3. DOckerFile: file instruction set, demonstrate how to auto create Docker Images

[Docker Structure]()

#### Support by OS
1. Namespaces: first level of isolation
2. Control Groups: a part of LXC(Linux Control), an OS level virtualization method for running multiple isolated Linux systems(containers)
3. UnionFS: file system, create user layer

