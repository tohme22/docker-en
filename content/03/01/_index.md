---
title: "Demo"
description: "docker"
draft: false
weight: 2
---
### Comamnds used

```yaml
# List all Docker images available locally
$ docker image ls

# Download the Ubuntu image from Docker Hub
$ docker pull ubuntu

# Create and run an Ubuntu-based container to execute the 'ls' command
$ docker run ubuntu ls

# Delete a container (replace <container_name> with the container's name)
$ docker rm <container_name>

# Create and run an interactive container named 'ubuntu_container' based on Ubuntu
$ docker run -it --name ubuntu_container ubuntu

From another terminal:
---------------------------
# Display real-time resource usage statistics (CPU, memory, etc.) for containers
$ docker stats 

Return to the first terminal:
------------------------------
# Exit the container's shell
root@9dad023013d9:/# exit

# Rename the container ubuntu_container to ubuntu_cont
$ docker rename ubuntu_container ubuntu_cont

# List all containers (active and stopped)
$ docker ps -a
```

```yaml
# Run an interactive container based on the Node.js image
$ docker run -it node 

# Perform an addition in the Node.js REPL
> 1 + 1
2
# Exit the Node.js REPL
> .exit

# Run an interactive container based on the Python image
$ docker run -it python 

# Perform an addition in the Python interpreter
>>> 1 + 1
2
# Exit the Python interpreter
>>> exit()

# List all containers (active and stopped)
$ docker ps -a

# Start a container (replace <container_name> with the container's name)
$ docker start <container_name>

# Start a container in interactive mode
$ docker start -i <container_name>

# Stop a running container
$ docker stop <container_name>

# Forcefully stop a container immediately (kill)
$ docker kill <container_name>

# Display real-time usage statistics (CPU, memory, etc.)
$ docker stats 

# Display logs of a container
$ docker logs <container_name>

# Display detailed information about a container in JSON format
$ docker inspect <container_name>

# Run a Redis container in the background
$ docker run -d redis

# Open a redis-cli session inside an existing container (replace <container_name>)
$ docker exec -it <container_name> redis-cli 

# Exit the redis-cli prompt
127.0.0.1:6379> exit

# List all containers
$ docker ps -a

# Stop a container
$ docker stop <container_name>
```

```yaml
# Run a Jupyter TensorFlow Notebook container and map port 12345 to 8888
$ docker run --rm -p 12345:8888 jupyter/tensorflow-notebook

Open a web browser:
-------------------------------
# Access Jupyter Notebook in the browser
http://127.0.0.1:12345
# Copy the token displayed in the Docker terminal and paste it into the browser
```
```yaml
# Stop all running and stopped containers
$ docker stop $(docker ps -a -q)

# Delete all containers
$ docker rm $(docker ps -a -q)

# Delete all Docker images
$ docker rmi $(docker images -a -q)

# Delete only images not used by any container
$ docker image prune -a

# Clean up all unused Docker resources to free up disk space
$ docker system prune
```
