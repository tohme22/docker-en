---
title: "Exercise"
description: "docker"
draft: false
weight: 3
---
### Exercise

1- Search the website https://hub.docker.com/ the application **jupyter/tensorflow-notebook** to verify that it exists.

2-  Execute the command:
```yaml
$ docker run jupyter/tensorflow-notebook
```

3- The container will be created, but you will not have access because we need to use the `-p` option with `docker run` to open a port to access the local jupyter website.

4- Enter `CTRl+C` to stop the container.

5- Execute the command:
```yaml
$ docker run -p 12345:8888 jupyter/tensorflow-notebook
```

6- Open a browser and type: **http://127.0.0.1:12345**

7-  Copy the **token** from the command line and paste it into the opened web page.

![](../../images/ex1a.png?height=150&classes=border,shadow,inline)

8- You should have this page open:

![](../../images/ex1.png?height=400&classes=border,shadow,inline)

9- Click on **Notebook --> Python 3 (ipykernel)**

10- Type this command to test the container:

`!pip freeze`

Then click on the run button, to test. You should have this result.

Note: The `!pip freeze` command in a Jupyter Notebook (or any Python environment) lists all the installed Python packages along with their versions.

![](../../images/ex1b.png?height=400&classes=border,shadow,inline)

11- Perform another test. Close the Notebook and click on **Terminal**

![](../../images/ex1c.png?height=150&classes=border,shadow,inline)

12- Try some commands:

![](../../images/ex1d.png?height=200&classes=border,shadow,inline)

13- Go back to your Linux terminal and open a new tab. Type the command `sudo netstat -tunap | grep docker` to display the connection between the host and the container.

![](../../images/ex1e.png?height=200&classes=border,shadow,inline)

14- Without stopping the current container, run another container using the same image:

```yaml
$ docker run -p 12345:8888 jupyter/tensorflow-notebook
```
It will give an **error message** because it is trying to use the same TCP port.

15- Use anothrer port, ex: `7777`

```yaml
$ docker run -p 7777:8888 jupyter/tensorflow-notebook
```

16- Check that you can access this new container using the browser.

17- Type the command `sudo netstat -tunap | grep docker` to display the connection between the host and the two containers.

18- Show running containers:

```yaml
$ docker ps -a
```

19- Check the status of the docker using the command `docker stats` 

19- Using the command `docker stop` stop the container that uses the port `12345`.

20- Delete the container that uses the port `12345`.

21- Run a new container using the port `8080`, but this time use the `–rm` option so that the container automatically removes itself after it stops.

```yaml
$ docker run --rm -p 8080:8888 jupyter/tensorflow-notebook
```

22- Show running containers `docker ps -a` and notice the new container with port `8080`.

23- Stop the new container and run again `docker ps -a`, the new container has been deleted automatically.

24- Stop the rest of the containers and delete all the containers using the command `docker rm $(docker ps -a -q)`. Confirm with the command`docker ps -a`.

25- List all images `docker image ls` and delete the image `docker image rm jupyter/tensorflow-notebook`.
