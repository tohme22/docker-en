---
title: "8. Docker Compose"
description: "docker"
draft: false
weight: 8
---
We first learned how to run a docker container using the docker run command. What if we need to set up a complex application running multiple services?

### What is Docker compose?

 **Docker Compose** is used to run multiple containers as a single service. For example, lets imagine you have  an application which required NGNIX and MySQL, with docker compose you could create one configuration  file (_in `yaml format`_)  called `dockercompose.yml` which would start both the containers as a service without the need to start each one separately.
 
**https://docs.docker.com/compose/compose-file/compose-file-v3/**

A minimal Docker Compose application consists of three components:

1. A `Dockerfile` for each container image you want to build.
2. A YAML file, `docker-compose.yml`, that Docker Compose will use to launch containers from those images and configure their services.
3. The files that comprise the application itself.

### Docker compose file

As explainded, Docker Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration

Here is a simple docker-compose.yml file:
```yaml
# docker-compose.yml
version: '3'

services:
  web:
    build: .
    # build from Dockerfile
    context: ./Path
    dockerfile: Dockerfile
    ports:
     - "5000:5000"
    volumes:
     - .:/code
  redis:
    image: redis
```

### Common commands

- **docker-compose start** Starts existing containers for a service.
- **docker-compose stop** Stops running containers without removing them.
- **docker-compose pause** Pauses running containers of a service.
- **docker-compose unpause** Unpauses paused containers of a service.
- **docker-compose ps** Lists containers.
- **docker-compose up** Builds, (re)creates, starts, and attaches to containers for a service.
- **docker-compose down** Stops containers and removes containers, networks, volumes, and images created by up.
---

### Config file reference

#### Building
```yaml
web:
  # build from Dockerfile
  build: .
  # build from custom Dockerfile
  build:
    context: ./dir
    dockerfile: Dockerfile.dev
  # build from image
  image: ubuntu
  image: ubuntu:14.04
  image: tutum/influxdb
  image: example-registry:4000/postgresql
  image: a4bc65fd
```

#### Ports
```yaml
ports:
  - "3000"
  - "8000:80"  # guest:host
# expose ports to linked services (not to host)
expose: ["3000"]
```

#### Commands
```yaml
# command to execute
command: bundle exec thin -p 3000
command: [bundle, exec, thin, -p, 3000]

# override the entrypoint
entrypoint: /app/start.sh
entrypoint: [php, -d, vendor/bin/phpunit]
```

#### Environment variables
```yaml
# environment vars
environment:
  RACK_ENV: development
environment:
  - RACK_ENV=development

# environment vars from file
env_file: .env
env_file: [.env, .development.env]
```

####  Dependencies
```yaml
# makes the `db` service available as the hostname `database`
# (implies depends_on)
links:
  - db:database
  - redis

# make sure `db` is alive before starting
depends_on:
  - db
```


#### Other options
```yaml
# make this service extend another
extends:
  file: common.yml  # optional
  service: webapp
volumes:
  - /var/lib/mysql
  - ./_data:/var/lib/mysql
```
---

### Advanced features

#### Labels
```yaml
services:
  web:
    labels:
      com.example.description: "Accounting web app"
```

#### DNS servers
```yaml
services:
  web:
    dns: 8.8.8.8
    dns:
      - 8.8.8.8
      - 8.8.4.4
```

#### Devices
```yaml
services:
  web:
    devices:
      - "/dev/ttyUSB0:/dev/ttyUSB0"
```

#### External links
```yaml
services:
  web:
    external_links:
      - redis_1
      - project_db_1:mysql
```

#### Hosts
```yaml
services:
  web:
    extra_hosts:
      - "somehost:192.168.1.100"
```

#### Network
```yaml
# creates a custom network called `frontend`
networks:
  frontend:
```

#### External network
```yaml
# join a preexisting network
networks:
  default:
    external:
      name: frontend
```
---

References: 

[https://www.tutorialspoint.com/docker/docker\_compose.htm](https://www.tutorialspoint.com/docker/docker_compose.htm)

[https://www.infoworld.com/article/3254689/docker-tutorial-get-started-with-docker-compose.html](https://www.infoworld.com/article/3254689/docker-tutorial-get-started-with-docker-compose.html)

[https://www.educative.io/blog/docker-compose-tutorial](https://www.educative.io/blog/docker-compose-tutorial)

[https://miro.medium.com/max/3116/1\*jLReq3Ng6RhqzJo4kBPnpA.png](https://miro.medium.com/max/3116/1*jLReq3Ng6RhqzJo4kBPnpA.png)

[https://docs.docker.com/compose/compose-file/compose-versioning/](https://docs.docker.com/compose/compose-file/compose-versioning/)

[https://sreeninet.wordpress.com/2017/03/28/comparing-docker-compose-versions/](https://sreeninet.wordpress.com/2017/03/28/comparing-docker-compose-versions/)

.







