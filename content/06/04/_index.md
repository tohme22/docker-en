---
title: "Solutions"
description: "$ docker"
draft: true
weight: 4
---
### Solution - Exercise 1

1.	 Run a named container `webserver` with a simple web server `nginx` using the `host`.
```yaml
$ docker run -d --network host --name webserver nginx
```

2.	Inspect the container's network configuration.
```yaml
$ docker inspect webserver
```

3.	Access the web server from the host machine using a browser.
```yaml
$ http://localhost
```

4. Create a new Bridge network named `bridge1` with the subnet address `192.168.3.0/24` and the `gateway=192.168.3.1`.
```yaml
$ docker network create --driver bridge --subnet=192.168.3.0/24 --gateway=192.168.3.1 bridge1
```

5. Run another container, named `webserver-bridge` with the same web server, using the new network `bridge1`.
```yaml
$ docker run -d --name webserver-bridge --network bridge1 nginx
```

6.	Inspect the network configuration of this new container.
```yaml
$ docker inspect webserver-bridge
```

7.	Access the new web server from the host machine using a browser.
```yaml
$ http://192.168.3.2
```

8. List Docker networks.
```yaml
$ docker network list
```

9. Stop the two containers created in this exercise.
```yaml
$ docker stop webserver webserver-bridge 
```

10. Try to remove the host network. Are you able? Why?
```yaml
$ docker network rm host
Error response from daemon: host is a pre-defined network and cannot be removed
```

11. Delete the network `bridge1`.
```yaml
$ docker network rm bridge1 
```

12. Start the container `webserver-bridge`. Are you able? Why?
```yaml
$ docker start webserver-bridge 
Error response from daemon: network e678188caaa2c94c4949f6cee54d5ef4712c8a27e064169e6b2971ffc98dd592 not found
Error: failed to start containers: webserver-bridge
```

13. Fix the problem so that you can start the existing container `webserver-bridge` with the default network `bridge` not `bridge1`.
```yaml
$ docker network disconnect bridge1 webserver-bridge
$ docker start webserver-bridge 
```

14.	Inspect the container's network configuration `webserver-bridge`.
```yaml
$ docker inspect webserver-bridge
```

15. Stop and delete the containers created during this exercise. Confirm that both containers have been deleted.
```yaml
$ docker stop webserver-bridge 
$ docker rm webserver webserver-bridge 
$ docker ps -a
```
---

### Solution - Exercise 2

1.	Create a network `macvlan` with the following info: `192.168.10.0/24 Gateway 192.168.10.1` and attach it to the physical interface of your host `ens160`.
```yaml
$ docker network create -d macvlan --subnet=192.168.10.0/24 --gateway=192.168.10.1 -o parent=ens160 my_macvlan
```

2.	Open two terminals. On each terminal, launch a container `alpine` using the new network `macvlan` and assign a **static IP addresses** for each. `192.168.10.5` for the 1st container and `192.168.10.6` for the 2nd container.
```yaml
$ docker run --rm -it --network my_macvlan --ip 192.168.1.5 --name container1 alpine
$ docker run --rm -it --network my_macvlan --ip 192.168.1.6 --name container2 alpine
```

3.	Verify the IP addresses assigned of each container..
```yaml
/ # ip ad
inet 192.168.10.5/24 brd 192.168.10.255 scope global eth0

/ # ip ad
inet 192.168.10.6/24 brd 192.168.10.255 scope global eth0
```

4. Test network connectivity between the two containers using `ping`. What do you observe?
```yaml
/ # ping 192.168.10.5
PING 192.168.10.5 (192.168.10.5): 56 data bytes
64 bytes from 192.168.10.5: seq=0 ttl=64 time=0.265 ms
64 bytes from 192.168.10.5: seq=1 ttl=64 time=0.081 ms
...
```

5.	Clean up by removing containers and macvlan network.
```yaml
$ docker stop container1 container2
$ docker stop container1 container2
$ docker network rm my_macvlan




