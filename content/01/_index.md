---
title: "1. Basic Concepts"
description: "docker"
draft: false
weight: 1
---
### Containers

The container isolate a **process** using **namespaces** and **control groups (cgroups)** in Linux, which are features of the Linux kernel.

- Namespaces provide a layer of isolation: each aspect of a container runs in its own namespace and does not have access outside of it. 
- Control groups limit and distribute the hardware resources like CPU and memory to the containers.

![](../images/nmpaces.png?height=400&classes=border,shadow,inline)

Containers are like very lightweight virtual machines that can run directly on our host operating system's kernel without the need of a hypervisor. As a result we can run multiple containers simultaneously.

So, while containers can give an illusion of a lightweight OS due to their isolation and self-contained nature, they don't actually contain a separate OS kernel like a virtual machine would. 

They use the host's kernel and merely include the application and its immediate dependencies, which can include libraries, binaries, etc., needed to run the application

![](../images/container-what-is-container.jpg?height=500&classes=border,shadow,inline)

Each container contains an application along with all of its dependencies and is isolated from the other ones. Developers can exchange these containers as image\(s\) through a registry and can also deploy directly on servers.

### Comparing Virtual Machines and Containers

![](../images/container-vm-container.jpg)

**Virtual machines:**

* A virtual machine is the emulated equivalent of a physical computer system with their virtual CPU, memory, storage, and operating system.
* A program known as a hypervisor creates and runs virtual machines. The physical computer running a hypervisor is called the host system, while the virtual machines are called guest systems.
* The hypervisor treats resources — like the CPU, memory, and storage — as a pool that can be easily reallocated between the existing guest virtual machines.

> **There are two types of hypervisors:**
>
> **Type 1 Hypervisor** \(VMware vSphere, KVM, Microsoft Hyper-V\). 
>
> **Type 2 Hypervisor** \(Oracle VM VirtualBox, VMware Workstation Pro/VMware Fusion\).

**Containers:**

* A container is an abstraction at the application layer that packages code and dependencies together. 
* Instead of virtualizing the entire physical machine, containers virtualize the host operating system only.
* Containers sit on top of the physical machine and its operating system. Each container shares the host operating system kernel and, usually, the binaries and libraries, as well.

| What's Diff? | VMs | Containers |
| :--- | :--- | :--- |
| size | Heavyweight \(GB\) | Lightweight\(MB\) |
| BootTime | Startup time in minutes | Startup time in seconds |
| Performance | Limited performance | Native performance |
| OS | Each VM runs in its own OS | All containers share the host OS |
| Runs on | Hardware-level virtualization\(Type1\) | OS virtualization |
| Memory | Allocates required memory | Requires less memory space |
| Isolation | Fully isolated and hence more secure | Process-level isolation, possibly less secure |

### Docker

Docker is an open source containerization platform. It provides the ability to run applications in an isolated environment known as a container.

### Installing Docker

#### Red Hat , Fedora, Centos, AlmaLinux, Rocky Linux, Oracle Linux

**SET UP THE REPOSITORY**

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

```yaml
$ sudo dnf install -y epel-release
$ sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
**INSTALL DOCKER ENGINE**

Install the _latest version_ of Docker Engine and containerd, or go to the next step to install a specific version:

```yaml
$ sudo dnf -y update
$ sudo dnf install -y docker-ce
```

**START DOCKER**

```yaml
$ sudo systemctl enable --now docker
$ docker 
```

**OPTIONAL: INSTALL DOCKER-COMPOSE**

**Docker-compose** is used to create multi-conteners applications.

```yaml
$ wget https://github.com/docker/compose/releases/download/v2.23.3/docker-compose-linux-x86_64
$ sudo mv docker-compose-linux-x86_64 docker-compose
$ sudo chmod +x docker-compose
$ sudo mv docker-compose /usr/local/bin/
$ docker-compose

```
#### Debian, Ubuntu, Kali

**INSTALL DOCKER ENGINE & DOCKER-COMPOSE**

Update the `apt` package index, and install the _latest version_ of Docker Engine , or go to the next step to install a specific version:

```yaml
 $ sudo apt update
 $ sudo apt install -y docker.io docker-compose
 $ systemctl status docker
```

### Windows 10/11

The installation steps are as follows:

1- Navigate to this site : https://learn.microsoft.com/en-us/windows/wsl/install and follow the instructions for installing **WSL2** on Windows 10.

2- Then navigate to the official download page https://www.docker.com/products/docker-desktop/ and click the Download for Windows (stable) button.

3- Double-click the downloaded installer and go through the installation with the defaults.

Once the installation is done, start ***Docker Desktop*** either from the start menu or your desktop. The docker icon should show up on your taskbar.

![](../images/docker-icon-in-taskbar.png?height=100&classes=border,shadow,inline))

Now, open up Ubuntu or whatever distribution you've installed from Microsoft Store. Execute the **docker --version** and **docker-compose --version** commands to make sure that the installation was successful.

![](../images/wsl-docker.png?height=&classes=border,shadow,inline))

### Hello World in Docker

Now that we have Docker ready to go on our machines, it's time for us to run our first container. Open up terminal and run following command:

```yaml
$ sudo docker run hello-world
```
If everything goes fine you should see some output like the following:

```yaml
[root@earth]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete 
Digest: sha256:49a1c8800c94df04e9658809b006fd8a686cab8028d33cfba2cc049724254202
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

### Docker Hub

![](../images/docker-hub.png?height=400&classes=border,shadow,inline))


The **Docker Hub**is a public repository maintained by Docker Inc.

**https://hub.docker.com/**

It contains a large number of images (official or not) that can be downloaded forinstanciercontainers

- Nginx, Redis, MariaDB, Wordpress,…

It is possible to create an account:

- To publish images to the public repository

- To create a private repository with private image

### Docker Engine

To understand what just happened, you need to get familiar with the Docker Architecture, Images and Containers, and Registries.

Docker Engine is a client-server application with these major components:
- A **server** which is a type of long-running program called a daemon process (the dockerd command).
- A **REST API** which specifies interfaces that programs can use to talk to the daemon and instruct it what to do.
- A **command line interface (CLI)** client (the docker command).

![](../images/container-docker-engine.jpg?height=350&classes=border,shadow,inline))

### Docker Architecture

Docker’s architecture is also client-server based. However, it’s a little more complicated than a virtual machine because of the features involved. It consists of four main parts:

![](../images/container-docker-arch.jpg?height=400&classes=border,shadow,inline))

1. **Docker Client**: This is how you interact with your containers. Call it the user interface for Docker.
2. **Docker Objects**: These are your main components of Docker: your containers and images. We mentioned already that containers are the placeholders for your software, and can be read and written to. Container images are read-only, and used to create new containers.
3. **Docker Daemon**: A background process responsible for receiving commands and passing them to the containers via command line.
4. **Docker Registry**: Commonly known as Docker Hub, this is where your container images are stored and retrieved.

>When you are working with Docker, you use images, containers, volumes, networks; all these are **Docker objects**.

--------
Ressources:

[https://www.docker.com/resources/what-container](https://www.docker.com/resources/what-container)

[https://www.backblaze.com/blog/vm-vs-containers/](https://www.backblaze.com/blog/vm-vs-containers/)

[https://cloudacademy.com/blog/docker-vs-virtual-machines-differences-you-should-know/](https://cloudacademy.com/blog/docker-vs-virtual-machines-differences-you-should-know/)

[https://docs.docker.com/get-started/overview/](https://docs.docker.com/get-started/overview/)

[https://docs.docker.com/engine/install/fedora/](https://docs.docker.com/engine/install/fedora/)

[https://docs.docker.com/engine/install/centos/](https://docs.docker.com/engine/install/centos/)

[https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)


