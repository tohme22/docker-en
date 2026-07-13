---
title: "Dockerfile"
description: "docker"
draft: false
weight: 2
---
### Docker File

- You might **create your own images** or you might only use those created by others and published in a registry. 
- To **build your own image**, you create a **`Dockerfile`** with a simple syntax for defining the steps needed to create the image and run it. 
- Each instruction in a Dockerfile **creates a layer** in the image. 
- When you change the Dockerfile and rebuild the image, only those layers which have changed are rebuilt. 
- A **Dockerfile** is executed by the **`docker build`** command.

Lets take a look at sample Dockerfile:

```yaml
#Nginx
#
# VERSION    0.0.1
FROM    ubuntu
LABEL Description="test image" Vendor="ACME Products"
RUN apt-get update && apt-get install -y nginx openssh-server
```

#### Dockerfile instructions:

* **FROM**: defines the base image; the FROM instruction must be the first instruction in Dockerfile.
* **LABEL**: describes any thing you want to define about this image.
* **WORKDIR**: defines the working directory of the container.
* **COPY**: copies files into the image, preferred over ADD.
* **ADD**: copies a file into the image but supports tar and remote URL.
* **VOLUME**: creates a mount point as defined when the container is run.
* **RUN**:  runs a new command in a new layer.
* **CMD**: specifies the default command and/or parameters for the container at startup.
* **ENTRYPOINT**: sets the command that will be executed when the container starts. 
* **EXPOSE**: documents the ports that should be published.
* **ENV**: defines environmental variables in the container.
* **ARG**: defineds variables used during the creation of the image (use with `--build-arg`)
* **MAINTAINER**: documents the author of the Dockerfile \(typically an email address\)
* **ONBUILD**: use as a trigger when this image is used to build other images; will define commands to run "on build"

Now to build an image from this Dockerfile, we'll use the build command. The generic syntax for the command is as follows:

```yaml
docker build -t <build context>
```

The build command requires a Dockerfile and the build's context. The context is the set of files and directories located in the specified location. Docker will look for a Dockerfile in the context and use that to build the image.

Open up a terminal window inside that directory and execute the following command:

```console
[root@earth]# ls
dockerfile

[root@earth]# docker build .
```

>Do not use your root directory, `/`, as the `PATH` as it causes the build to transfer the entire contents of your hard drive to the Docker daemon.

We're passing `.` as the build context which means the current directory. If you put the Dockerfile inside another directory like /src/Dockerfile, then the context will be `./src`. The build process may take some time to finish:

```console
[root@earth]# docker build .
Sending build context to Docker daemon  14.85kB
Step 1/3 : FROM ubuntu
 ---> adafef2e596e
Step 2/3 : LABEL Description="test image" Vendor="ACME Products"
 ---> Running in 043a4ae90eb9
Removing intermediate container 043a4ae90eb9
 ---> bdeb04e9a931
Step 3/3 : RUN apt-get update && apt-get install -y nginx openssh-server
 ---> Running in ae0d0c698212
Get:1 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:2 http://security.ubuntu.com/ubuntu focal-security InRelease [107 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal-updates InRelease [111 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal-backports InRelease [98.3 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [11.3 MB]
Get:6 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [187 kB]
Get:7 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [1078 B]
.
.
.
Successfully built fc32da11d651
```

#### Dockerfile Build Oprions

* **--file, -f:** Specifies the name and path of the Dockerfile to use for the build. This is useful if your Dockerfile is named something other than the default Dockerfile or is not located in the root of the context directory.

* **--tag, -t:** Names and optionally tags the built image. This is useful for assigning a name and tag to your new image, which makes it easier to manage and use later.

* **--build-arg key=value:** Allows you to pass variables at build time that can be accessed like environment variables in the Dockerfile. It's useful for passing secrets or environment-specific settings that shouldn't be hard-coded in the Dockerfile.

* **--no-cache:** Disables the cache when building the image. Without this, Docker uses a cache of previous builds to speed up the build process. This is useful if you want to ensure that every step of the Dockerfile is executed fresh, often used for troubleshooting or ensuring up-to-date content.

* **--cache-from:** Allows specifying images to consider as cache sources. This is useful for speeding up builds by reusing data from previous builds or other specified images.

* **--pull:** Always attempts to pull a newer version of the base and parent images before starting the build. This ensures that you are using the most up-to-date version of base images.

* **--rm:** Removes intermediate containers after a successful build by default. This flag is enabled by default and can be disabled with --no-rm to troubleshoot or inspect intermediate containers.

* **--squash:** Squashes the newly built layers into a single new layer on top of the base image. This can help reduce the size of the resulting image, but it's experimental and requires Docker daemon experimental features to be enabled.

* **--add-host:** Adds a custom host-to-IP mapping (host:ip) in the build environment. This is useful for adding additional entries to /etc/hosts in the build environment.

* **--network:** Sets the networking mode for the RUN instructions during build. This can be useful for specifying the network settings, such as using host networking or a custom network.

* **--platform:** Specifies the platform of the image to build, in the format os[/arch[/variant]]. This option is useful for building images for different hardware architectures from the one you are currently running.

* **--secret:** Allows passing secret information for use during the build, without including it in the final image. This is useful for sensitive data like passwords or API keys that the build might need access to but should not be in the image.

* **--ssh:** Enables forwarding SSH agent connections. Useful for installing private packages or accessing private repositories during the build without leaving SSH keys in the final image.

### Dockerfile examples

#### Example 1
In this example, we'll create a Docker image based on `Nginx version 1.10.1` using `Alpine Linux` as the base operating system. `Dockerfile` will copy a file `index.html`from the source folder to the folder `/usr/share/nginx/html` inside the image, allowing for customization of the home page served by Nginx. The Dockerfile also specifies that the container listens on port `8080`, but note that this does not publish the port automatically; this must be done when the container is launched. Finally, it configures Nginx to run in mode `daemon off`, which is necessary to keep the container running, as Docker requires the main process (PID 1) to remain alive so that the container does not terminate.

```yaml
$ mkdir nginx-docker
$ cd nginx-docker

$ vim index.html
  <!DOCTYPE html>
  <html>
    <head>
      <title>Ma nouvelle image</title>
    </head>
    <body>
      <h1>Mon site Web !</h1>
    </body>
  </html>

$ vim dockerfile
  # Uses the base Nginx image version 1.10.1 based on Alpine Linux. This creates a lightweight image containing the Nginx web server.
  FROM nginx:1.10.1-alpine

  # Copies the index.html file from the source folder to the /usr/share/nginx/html folder in the image. This allows displaying your custom web page via Nginx.
  COPY index.html /usr/share/nginx/html

  # Informs Docker that the container listens on port 8080 at runtime. This does not publish the port by itself but serves as documentation between the image creator and the end user.
  EXPOSE 8080

  # Sets the default command that will be executed when the container starts. Here, it starts the Nginx server in foreground mode so that the container does not stop immediately after starting.
  CMD ["nginx", "-g", "daemon off;"]


$ docker build -t atohme/nginx-alpine:latest .
$ docker image ls
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
atohme/nginx-alpine   latest    93389ccd2c1c   10 seconds ago   54MB

$ docker run -d -p 8080:80 atohme/nginx-alpine
$ docker ps -a
$ curl http://127.0.0.1:8080
$ docker exec -it <nom_container> /bin/sh
```

#### Example 2
Let's create a `Dockerfile` for an application based on `Python`. This example will show how to set up a simple Python environment with `Flask` which is a popular web framework. This will create a basic Flask application that responds with “Hello, World!” by accessing the root URL. 

```yaml
$ mkdir my-flask-app 
$ cd my-flask-app

$ vim requirements.txt
  Flask==2.0.1
  Werkzeug==2.0.1

$ vim app.py
  from flask import Flask
  app = Flask(__name__)
  @app.route('/')
  def hello_world():
      return 'Hello, World!'
  if __name__ == '__main__':
      app.run(debug=True, host='0.0.0.0') 

$ vim dockerfile
  # Uses an official Python image as the parent image.
  FROM python:3.9-slim
  
  # Sets the working directory in the container to /app
  WORKDIR /app
  
  # Copies the content of the current directory to /app in the container
  COPY . /app
  
  # Copies the requirements.txt file to /app
  COPY requirements.txt /app/
  
  # Installs the necessary packages specified in requirements.txt
  RUN pip install --no-cache-dir -r requirements.txt

  # Makes port 5000 available to the outside world from this container
  EXPOSE 5000
  
  # Sets an environment variable
  ENV NAME World
  
  # Runs app.py when the container starts
  CMD ["python", "app.py"]

$ docker build -t atohme/my-flask-app:1.0 .
$ docker run -p 5000:5000 atohme/my-flask-app:1.0
```

![](../../images/image1.png?height=&classes=border,shadow,inline))


##### Rename the image tag :
```yaml
$ docker tag atohme/my-flask-app:1.0 atohme/my-flask-app:2.0
$ docker image ls
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
atohme/my-flask-app   1.0       ed4c17a7660a   2 minutes ago    137MB
atohme/my-flask-app   2.0       ed4c17a7660a   2 minutes ago    137MB

$ docker rmi atohme/my-flask-app:1.0
Untagged: atohme/my-flask-app:1.0

$ docker image ls
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
atohme/my-flask-app   2.0       ed4c17a7660a   2 minutes ago    137MB
```

#### Exemple 3
This example will help you demonstrate how to set up an environment `Node.js`, manage dependencies, and run a basic web server with `Express`, a popular `Node.js` framework. This setup creates a Node.jsbasic application that responds with `“Hello World!”` when the root URL is accessed.

```yaml
$ mkdir node-express
$ cd node-express

$ vim dockerfile
  # Uses the official Node.js 14 image as the parent image
  FROM node:14
  # Sets the working directory in the container to /app
  WORKDIR /app
  # Copies the package.json and package-lock.json files to the working directory
  COPY package*.json ./
  # Installs the Node.js dependencies defined in package.json
  RUN npm install
  # Copies the rest of your application's source code to the container
  COPY . .
  # Informs Docker that the container listens on port 3000 at runtime
  EXPOSE 3000
  # Sets the command to run your application
  CMD [ "node", "server.js" ]
 
$ vim package.json
  {
    "name": "nodejs-app",
    "version": "1.0.0",
    "main": "server.js",
    "dependencies": {
       "express": "^4.17.1"
   },
    "scripts": {
      "start": "node server.js"
   }
  }

$ vim server.js
  const express = require('express');
  const app = express();
  const port = 3000;
  app.get('/', (req, res) => {
  res.send('Hello World!');
  });
  app.listen(port, () => {
    console.log(`Example app listening at http://localhost:${port}`);
  });

$ docker build -t node-express:1.0 .
$ docker image ls
$ docker run -p 3000:3000 node-express:1.0
```
![](../../images/image2.png?height=&classes=border,shadow,inline))