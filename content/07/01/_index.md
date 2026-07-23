---
title: "Dockerfile"
description: "docker"
draft: false
weight: 2
---

# Dockerfile Guide

- You can **create your own images** or simply use those created by others and published in a registry.
- To **build your own image**, you create a **`Dockerfile`** with a simple syntax to define the steps required to create the image and run it.
- Each instruction in a Dockerfile **creates a layer** in the image.
- When you modify the Dockerfile and rebuild the image, only the modified layers are rebuilt.
- A **Dockerfile** is executed using the **`docker build`** command.

---

## Steps to Build an Image

### Example 1
- Create an image based on `Ubuntu Linux` and `Nginx version 1.10.1`.
- The Dockerfile copies an `index.html` file into `/usr/share/nginx/html` and exposes port `8080`.

**1. Create a working directory:**

```console
[root@earth]# mkdir nginx-docker
[root@earth]# cd nginx-docker
```

**2. Create additional files (e.g., scripts or web files):**

In this example, it is **`index.html`**:

```console
[root@earth-nginx-docker]# vim index.html
  <!DOCTYPE html>
  <html>
    <head>
      <title>My New Image</title>
    </head>
    <body>
      <h1>My Website!</h1>
    </body>
  </html>
```

**3. Create the `Dockerfile`:**

```console
[root@earth-nginx-docker]# vim Dockerfile
```

```dockerfile
# Dockerfile for an Nginx web server

# 1. FROM: choose a base Ubuntu image
FROM ubuntu:24.04

# 2. LABEL: add metadata description (optional but good practice)
LABEL maintainer="tohmea@itmt.ca" \
      description="Small Nginx server for Dockerfile demonstration"

# 3. ARG: variable available only during the build process
# Here, it specifies the website directory
ARG SITE_DIR=/var/www/html

# 4. ENV: persistent environment variable inside the container
ENV MESSAGE="Welcome to Docker!"

# 5. WORKDIR: set the default working directory
WORKDIR ${SITE_DIR}

# 6. RUN: execute a command during image build (install Nginx here)
RUN apt-get update && apt-get install -y nginx && rm -rf /var/lib/apt/lists/*

# 7. COPY: copy a local file into the image
COPY index.html ${SITE_DIR}/index.html

# 8. VOLUME: create a volume mount point to modify site files
VOLUME ["${SITE_DIR}"]

# 9. EXPOSE: document the HTTP port
EXPOSE 80

# 10. CMD: default command executed when the container starts
# This launches Nginx in the foreground so the container does not exit immediately.
CMD ["nginx", "-g", "daemon off;"]
```

#### Dockerfile Instructions:

* **FROM**: Defines the base image (must always be the first instruction).
* **LABEL**: Adds metadata to the image (description, author, version, etc.).
* **ARG**: Defines variables available only during the build process (used with `--build-arg`).
* **ENV**: Defines environment variables accessible inside the container.
* **WORKDIR**: Sets the default working directory inside the container.
* **RUN**: Executes a command during image creation (each `RUN` creates a new layer).
* **COPY**: Copies local files into the image (preferred over `ADD`).
* **ADD**: Copies files, but can also extract `.tar` archives or download files from a remote URL.
* **VOLUME**: Declares a directory as a mount point (persistent data during execution).
* **EXPOSE**: Documents the ports used by the container (e.g., 80 for HTTP).
* **CMD**: Defines the default command or parameters executed when the container starts.
* **ENTRYPOINT**: Defines the main executable run on container startup (`CMD` can supply default arguments to it).
* **ONBUILD**: Sets an instruction to be executed automatically if this image is used as a base for another image build.

**4. Build the new image:**

To build an image from this `Dockerfile`, use the **`docker build -t`** command.

```console
docker build -t <build context>
```

We pass **`.`** as the build context, which signifies the current directory.

```console
[root@earth-nginx-docker]# docker build -t tohmea/nginx-ubuntu:latest .
[root@earth-nginx-docker]# docker image ls
REPOSITORY              TAG       IMAGE ID       CREATED              SIZE
tohmea/nginx-ubuntu     latest    de8ec9c22f25   About a minute ago   84.6MB
```

**5. Create a container from this new image:**

```console
[root@earth-nginx-docker]# docker run -d --name nginx-tohmea -p 8080:80 tohmea/nginx-ubuntu
[root@earth-nginx-docker]# docker ps -a
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                                    NAMES
3f7a3503a650   tohmea/nginx-ubuntu   "nginx -g 'daemon of…"   5 seconds ago   Up 4 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   nginx-tohmea
```

**6. Test the container:**

```console
[root@earth-nginx-docker]# curl http://127.0.0.1:8080
<!DOCTYPE html>
<html>
  <head>
    <title>My new Image</title>
  </head>
  <body>
    <h1>My web site !</h1>
  </body>
</html>

[root@earth-nginx-docker]# docker exec -it nginx-tohmea bash
root@509cba1cca2a:/var/www/html# echo $MESSAGE
Welcome to Docker!
root@509cba1cca2a:/var/www/html# exit
```

---

### Example 2

Creating a `Python` environment with `Flask`.

**1. Create a working directory:**

```console
$ mkdir my-flask-app 
$ cd my-flask-app
```

**2. Create additional files:**

```text
$ vim requirements.txt

# Each line corresponds to a library to install via pip
# Required to run Flask with Python
Flask==2.0.1
Werkzeug==2.0.1
```

```python
$ vim app.py

# Example Flask Application (app.py)
# Import the Flask class from the flask module
from flask import Flask

# Create an instance of the Flask application
app = Flask(__name__)

# Define a route (URL "/")
# When a user accesses http://localhost:5000/,
# the hello_world() function will be executed
@app.route('/')
def hello_world():
    return 'Hello, World!'

# Main entry point of the application
if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0') 
```

**3. Create the `Dockerfile`:**

```dockerfile
$ vim Dockerfile

# Uses an official Python image as the parent image
FROM python:3.9-slim

# Sets an environment variable
ENV NAME="World"

# Sets the working directory inside the container to /app
WORKDIR /app

# Copies requirements files first (enables utilizing Docker layer cache)
COPY requirements.txt /app/

# Installs necessary packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Copies the rest of the source code into /app inside the container
COPY . /app

# Makes port 5000 accessible from outside this container
EXPOSE 5000

# Runs app.py when the container launches
CMD ["python", "app.py"]
```

**4. Build the image:**

```console
$ docker build -t tohmea/mon-flask-app:1.0 .
[+] Building 5.8s (11/11) FINISHED  docker:default
 => [internal] load build definition from dockerfile                                                                                    
 => => transferring dockerfile: 732B                                                                                                    
...
 => => naming to docker.io/tohmea/mon-flask-app:1.0     
```

**5. Create and run the container:**

```console
$ docker run -d -p 5000:5000 --name mon-flask tohmea/mon-flask-app:1.0
```

**6. Test:**

![Flask App Output](../../images/image1.png?height=&classes=border,shadow,inline)
