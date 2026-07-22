---
title: "Demo"
description: "docker"
draft: false
weight: 2
---
### Commands used

```yaml
# Download the Ubuntu image from Docker Hub
$ docker pull ubuntu

# List all Docker images available locally
$ docker image ls

# Create and run an Ubuntu container in interactive mode
$ docker run -it ubuntu

# List files inside the container (inside the container's root shell)
root$ ls

Open another Shell terminal:
--------------------------------
# List all containers (active and stopped)
$ docker ps -a        

# Stop a running container (replace container_name with the container's name)
$ docker stop container_name

# Delete a container (replace container_name with the container's name)
$ docker rm container_name

# Download the Ghost image from Docker Hub
$ docker pull ghost   

# Launch a Ghost container in the background with environment variables and port mapping
$ docker run -d --name ghost1 -e NODE_ENV=development -e url=http://localhost:3001 -p 3001:2368 ghost

Open a web browser:
-------------------------------
# Access the Ghost application via the browser
http://localhost:3001/

# List all containers (active and stopped)
$ docker ps -a  

# Stop the Ghost container named ghost1
$ docker stop ghost1

# Delete the Ghost container named ghost1
$ docker rm ghost1
```
