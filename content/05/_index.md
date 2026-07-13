---
title: "5. Docker Environment variables"
description: "docker"
draft: false
weight: 5
---

### Variables d’environnement 

**`-e`** **`--env`** --> Set environment variables. 

**`--env-file`** --> Read a variables file.

#### Demo 1: Using the -e option
```yaml
$ docker run --rm -it alpine
/ # echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
/ # echo $VAR1

/ # exit

$ docker run --rm -it -e VAR1="Bonjour Antoine" alpine
/ # echo $VAR1
Bonjour Antoine
/ # exit
```

#### Demo 2: Using the -e option

**https://jupyter-docker-stacks.readthedocs.io/en/latest/using/common.html**

```yaml
$ sudo mkdir /data
$ sudo touch /data/file.txt
$ docker run -p 8080:8888 -v /data:/home/jovyan/data jupyter/minimal-notebook

Open a browser:
---------------
http://127.0.0.1:8080
You will have an error : Permission denied if you try to creat a file.

Solution:
---------
$ docker run -it -v /data:/home/jovyan/data -p 8080:8888 -e CHOWN_HOME=yes -e NB_USER=jovyan --user root -w /home/$NB_USER -e CHOWN_HOME_OPTS='-R' jupyter/minimal-notebook
```
![](../images/env1.png?height=300&classes=border,shadow,inline)


#### Demo 3:  Using a env file
```yaml
$ cat envfile 
CHOWN_HOME=yes 
NB_USER=jovyan
CHOWN_HOME_OPTS='-R'

$ docker run -it -v /data:/home/jovyan/data -p 8080:8888 --env-file ./envfile --user root -w /home/$NB_USER jupyter/minimal-notebook
```
-------------------------

Ressources:

_https://docs.docker.com/compose/environment-variables/_
