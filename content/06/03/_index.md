---
title: "Exercises"
description: "$ docker"
draft: true
weight: 3
---
### Exercise 1 - Exploring the Host and Bridge networks in Docker

1. Run a named container `webserver` with a simple web server `nginx` using the `host` network.

2. Inspect the container's network configuration.

3. Access the web server from the host machine using a browser.

4. Create a new Bridge network named `bridge1` using the subnet address `192.168.3.0/24` and the `gateway=192.168.3.1`.

5. Run a container, named `webserver-bridge` with the web server `nginx`, using the new network `bridge1`.

6. Inspect the network configuration of this new container.

7. Access the new web server from the host machine using a browser.

8. List the Docker networks.

9. Stop the two containers created in this exercise.

10. Try to delete the host network. Are you able? Why?

11. Delete the network `bridge1`.

12. Start the container `webserver-bridge`. Are you able? Why?

13. Fix the problem so that you can start the existing container `webserver-bridge` with the default network `bridge` not `bridge1`.

14. Inspect the container's network configuration `webserver-bridge`.

15. Stop and delete the containers created during this exercise. Confirm that both containers have been deleted.
---

### Exercise 2 - Exploring Macvlan networks in Docker

1. Create a network `macvlan` with the following info: `192.168.10.0/24 Gateway 192.168.10.1` and attach it to the physical interface of your host `ens160`.

2. Open two terminals. On each terminal, launch a container `alpine` using the new network `macvlan` and assign a **static IP addresses** for each. `192.168.10.5` for the 1st container and `192.168.10.6` for the 2nd container.

3. Verify the IP addresses assigned to each container.

4. Test network connectivity between the two containers using `ping`. What do you observe?

5. Clean up by removing containers and macvlan network.





