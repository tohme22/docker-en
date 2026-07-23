---
title: "6. Docker Networking"
description: "docker"
draft: false
weight: 6
---

**https://docs.docker.com/network/**

Individual containers, need to  communicate with each other through a network to perform the required actions, and this is nothing but Docker Networking.

You can define Docker Networking as a communication passage through which all the isolated containers communicate with each other in various situations to perform the required actions.

### Overview of Docker Network Types

There are primarily **6 types of Docker networks**: **Bridge, Host, None, Overlay, IPvlan, and Macvlan**.

```yaml
# List supported Docker network types
[root@earth ~]# docker info | grep Network
Network: bridge host ipvlan macvlan null overlay

# Display the default docker0 bridge interface created by the Docker service
[root@earth ~]# nmcli
...
docker0: connected (externally) to docker0
        "docker0"
        bridge, 06:BC:99:0D:E3:77, sw, mtu 1500
        inet4 172.17.0.1/16
        route4 172.17.0.0/16 metric 0
...
```

> **Note:** The Docker daemon creates and configures the host's **docker0** interface as a **virtual Ethernet bridge** inside the Linux kernel. It is used by Docker containers to **communicate among themselves and with the outside world**. All Docker containers connect to **docker0** by default and utilize Docker's `iptables` NAT rules to reach external networks.

```yaml
# List default pre-defined Docker networks
[root@earth ~]# docker network ls
NETWORK ID        NAME        DRIVER      SCOPE
b05ff920f0da        bridge      bridge      local
f34113b7dc6a        host        host        local
ba9b9833188b        none        null        local
```

---

## 1. Default Bridge Network

The **Bridge** network is a *pre-defined private internal network* created by Docker on the host machine. Containers attached to this network receive an internal IP address and can communicate with each other via these IP addresses. Bridge networks are generally used when your applications run in standalone containers that need to intercommunicate.

![Bridge Network Diagram](../images/network1.png?height=350)

```yaml
# Inspect default bridge network details (subnet, gateway, attached containers, etc.)
$ docker network inspect bridge

...
"Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
...
```

### Example: Default Bridge Network

```yaml
# Create and start a container in the foreground on the default bridge network
$ docker run -it --rm --name alpine1 alpine
/ # ip add
2: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether a2:4c:fa:fb:67:1c brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
/ # exit

# Start a background container on the default bridge network
$ docker run -itd --rm --name demo1 busybox

# Start two additional containers on the default bridge network
$ docker run -itd --rm --name demo2 busybox
$ docker run -itd --rm --name demo3 busybox

# Inspect attached containers on the default bridge network
$ docker inspect bridge
"ConfigOnly": false,
        "Containers": {
            "610621fe0acc28ecac7074ecd4c926f7590ea10a9d9a7e41d1289a070f5ce844": {
                "Name": "demo2",
                "EndpointID": "e1e3cd0afb69b4712073bbbaa6089ba6ab8f74bdf4ca336482b3e1a31fdf12cf",
                "MacAddress": "92:05:c7:e2:ba:f7",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "91cfe2f84da4d06e534ea13b8fca6cddad6db7e146d432377c8791ecefc4634a": {
                "Name": "demo3",
                "EndpointID": "14341708beb214b16ed236c8f61b4de527acdfd8d4bf21e163f06e0ed272196b",
                "MacAddress": "26:10:81:f9:a7:8a",
                "IPv4Address": "172.17.0.4/16",
                "IPv6Address": ""
            },
            "f5e58a228a4c40a0e13a084740c4075e313213a80abf163203fe73e1b9532ed3": {
                "Name": "demo1",
                "EndpointID": "f3ad3dc259c60bb0642091718c4f12005f0367e38f551c4eeb3123dab141fa18",
                "MacAddress": "b6:05:f1:37:f2:6f",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },

# Display DNS configuration on the host
$ cat /etc/resolv.conf
nameserver 192.168.145.2
search localdomain

# Open an interactive shell inside demo1 container
$ docker exec -it demo1 sh

# Check network interfaces and IP address inside the container
/ # ip ad
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether b6:05:f1:37:f2:6f brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# Test connectivity to another container via IP address
/ # ping -c 3 172.17.0.3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.063 ms
```

### Shared DNS between Host and Default Bridge Network

```yaml
# Display DNS configuration inside container (inherited from host)
/ # cat /etc/resolv.conf
nameserver 192.168.145.2
search localdomain

# Display routing table inside container
/ # ip route
default via 172.17.0.1 dev eth0 
172.17.0.0/16 dev eth0 scope link  src 172.17.0.2

# Test DNS resolution and external internet connectivity
/ # ping -c 3 www.google.com
PING www.google.com (142.250.69.68): 56 data bytes
64 bytes from 142.250.69.68: seq=0 ttl=127 time=8.448 ms
...
/ # exit

# Stop and remove containers (automatically removed if --rm was passed)
$ docker stop demo1 demo2 demo3
$ docker ps -a
```

---

## Custom User-Defined Bridge Networks

### Example: Creating a User-Defined Bridge Network

![Custom Bridge Network Diagram](../images/network2.png?height=550)

```yaml
# Create a custom bridge network named net1
[root@earth ~]# docker network create --driver bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 net1

# List Docker networks
[root@earth ~]# docker network ls
NETWORK ID        NAME                DRIVER              SCOPE
b05ff920f0da        bridge              bridge              local
f34113b7dc6a        host                host                local
4c7a97493cf3        net1                bridge              local
ba9b9833188b        none                null                local

# Verify the new virtual bridge interface on host
[root@earth ~]# nmcli
br-9f0de78575a2: connected (externally) to br-9f0de78575a2
        "br-9f0de78575a2"
        bridge, 82:5A:A7:13:59:1E, sw, mtu 1500
        inet4 192.168.1.1/24
        route4 192.168.1.0/24 metric 0

# Inspect details of the new custom bridge network
[root@earth ~]# docker network inspect net1
    {
        "Name": "net1",
        "Id": "0f3495f1177c34e637d8091902ed7d1dd2191a8b9fe6d589c1bbf2c9adce5519",
        "Created": "2025-08-24T12:42:22.51811742-04:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.1.0/24",
                    "Gateway": "192.168.1.1"
...
```

### Running Containers on a User-Defined Bridge Network

```yaml
# Start app1 container on net1 network
[root@earth ~]# docker run -it --name app1 --network net1 alpine 
/ # ip ad
...
2: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:c0:a8:01:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.2/24 brd 192.168.1.255 scope global eth0
       valid_lft forever preferred_lft forever

# Press CTRL + P, followed by CTRL + Q to detach from container without stopping it.

# Start app2 container on net1 network and verify inter-container connectivity
[root@earth ~]# docker run -it --name app2 --network net1 alpine 
/ # ip ad
...
2: eth0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:c0:a8:01:03 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.3/24 brd 192.168.1.255 scope global eth0
       valid_lft forever preferred_lft forever

# Ping app1 from app2 using IP address
/ # ping -c 3 192.168.1.2
PING 192.168.1.2 (192.168.1.2): 56 data bytes
64 bytes from 192.168.1.2: seq=0 ttl=64 time=0.191 ms
...
```

### Embedded Docker DNS in User-Defined Bridge Networks

> **Note:** Containers on custom bridge networks can reach each other using container names. Docker provides an **embedded DNS server at 127.0.0.11** that automatically resolves container names. Containers also retain external internet access.

```yaml
# Inspect DNS configuration inside container on custom bridge
/ # cat /etc/resolv.conf
nameserver 127.0.0.11
search localdomain
options ndots:0

# Ping container by container name (Automatic DNS Resolution)
/ # ping -c 3 app1
PING app1 (192.168.1.2): 56 data bytes
64 bytes from 192.168.1.2: seq=0 ttl=64 time=0.087 ms

# Ping external domain name
/ # ping -c 3 www.google.com
PING www.google.com (142.250.69.68): 56 data bytes
64 bytes from 142.250.69.68: seq=0 ttl=127 time=11.729 ms
...

# Press CTRL + P, then CTRL + Q to detach.
```

Use `docker network inspect net1` to check active containers and assigned IP addresses on this network:

```yaml
[root@earth ~]# docker network inspect net1
 "Containers": {
            "2ca2b7353e439a27390d363237ba166e9bb9f957748ec01455a3f17a4fc14afa": {
                "Name": "app1",
                "EndpointID": "edfcfafd5da16efe1524ce744f8dae3cec57dc7cf6e80aa922e7b28d5bcebb2a",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "192.168.1.2/24",
                "IPv6Address": ""
            },
            "c353a85a7c70ff125815f817335abb98ebffa82f595878d20074aee8ed837f74": {
                "Name": "app2",
                "EndpointID": "a2e0b13be62ae9813ed3d70d423f32031b57a3cef30eec78598684f0459cbfdd",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "192.168.1.3/24",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
```

### Removing Custom Bridge Networks

```yaml
# Attempt to remove network with active endpoints attached
[root@earth ~]# docker network remove net1
Error response from daemon: error while removing network: network net1 id 4c7a97493cf3b2c151966afa5a933859ba6b246862f1d05ac9030d9aaa4d70ac has active endpoints

# Disconnect or force-remove attached containers before network deletion
[root@earth ~]# docker rm -f app1 app2
app1
app2

[root@earth ~]# docker network remove net1
net1

[root@earth ~]# docker network ls
NETWORK ID        NAME                DRIVER              SCOPE
b05ff920f0da        bridge              bridge              local
f34113b7dc6a        host                host                local
ba9b9833188b        none                null                local
```

> **Tip:** Running `docker network prune` will remove all unused custom networks at once.

---

## 2. Host Network

The **Host** network driver disables network isolation between the Docker host and containers, enabling containers to use the host's networking stack directly. Consequently, you cannot run multiple web containers listening on the same port on a single host, as port bindings are directly shared across all containers on the host network.

![Host Network Diagram](../images/network3.png?height=300)

With **host** networking, the container directly attaches to the host's physical network interface: **no NAT or port mapping is required.**

```yaml
# List available networks
[root@earth ~]# docker network ls
NETWORK ID     NAME        DRIVER    SCOPE
81dfee2cc1a8   bridge      bridge    local
24ca579cac93   host        host      local
5534e991653d   none        null      local

# Inspect configuration of the host network
[root@earth ~]# docker network inspect host 
    {
        "Name": "host",
        "Id": "24ca579cac936f740ef433ec9a2ff52798003db38790dab8492676d4eec0a05f",
        "Created": "2024-01-24T16:35:47.156057512-05:00",
        "Scope": "local",
        "Driver": "host",
...

# Attempting to remove the host network (impossible because it is pre-defined)
[root@earth ~]# docker network rm host
Error response from daemon: host is a pre-defined network and cannot be removed
```

### Usage Example: Host Network

```yaml
# Run an Nginx web server directly using the host's network stack
[root@earth ~]# docker run --rm -d --name web1 --network host nginx

[root@earth ~]# docker ps
CONTAINER ID   IMAGE   COMMAND                  CREATED         STATUS         PORTS    NAMES
50d79130ab8c   nginx   "nginx -g 'daemon of…"   5 seconds ago   Up 4 seconds            web1          
```

There is **no port mapping** (e.g., `-p 8080:80`) needed; port 80 is exposed directly on the host interface.

```yaml
# Identify host IP address
$ nmcli
ens160: connected to ens160
        "VMware VMXNET3"
        ethernet (vmxnet3), 00:0C:29:08:F5:AB, hw, mtu 1500
        ip4 default
        inet4 172.16.130.6/24
```

Open a web browser and navigate directly to **your Docker host's IP address**:

![Nginx Host Network Screenshot](../images/network-host-nginx.jpg?height=300&classes=border,shadow,inline)

---

## 3. MACvlan Network

**MACvlan** allows you to assign a **unique physical MAC address to each container**, making the container appear as a real physical network device (e.g., a physical PC or server) directly connected to the network.

- Containers **appear directly on the physical Local Area Network (LAN)** with their own dedicated **MAC and IP addresses**.
- To the physical network switch or router, **each container looks like an independent physical machine**.

MACvlan is ideal for:
- Applications that need to be **directly accessible from the physical network** without port translation.
- Systems requiring **minimal latency by bypassing bridge/NAT overhead** to achieve near line-rate physical hardware performance.

![MACvlan Network Diagram](../images/network4.png?height=500)

### Example: Creating a MACvlan Network

```yaml
# Identify parent interface, IP address, and default gateway of the host (e.g., enp0s3)
$ nmcli 
enp0s3: connected to enp0s3
        ethernet, 00:0C:29:08:F5:AB, hw, mtu 1500
        ip4 default
        inet4 10.7.1.232/24
        route4 10.7.1.0/24 metric 100
        route4 default via 10.7.1.2 metric 100

# Create MACvlan network matching physical host subnet and gateway
$ docker network create -d macvlan --subnet 10.7.1.0/24 --gateway 10.7.1.2 -o parent=enp0s3 macvlan1

# Verify macvlan1 creation
$ docker network ls
NETWORK ID     NAME        DRIVER    SCOPE
6b7447039910   bridge      bridge    local
f0230dc47a4e   host        host      local
829fffc0d9df   macvlan1    macvlan   local
7030aff8c1aa   none        null      local
```

### Running Containers on a MACvlan Network

```yaml
# Launch a container with a static IP on the macvlan1 network
$ docker run -it --rm --name demo1 --network macvlan1 --ip 10.7.1.200 busybox

# Verify assigned MAC address and IP address
/ # ip ad 
22: eth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether ce:03:3e:9b:f9:bf brd ff:ff:ff:ff:ff:ff
    inet 10.7.1.200/24 brd 10.7.1.255 scope global eth0

# Test connectivity with default network gateway
/ # ping -c 3 10.7.1.2
PING 10.7.1.2 (10.7.1.2): 56 data bytes
64 bytes from 10.7.1.2: seq=0 ttl=128 time=1.049 ms

# Test internet access
/ # ping -c 3 www.google.com
PING www.google.com (142.250.69.68): 56 data bytes
64 bytes from 142.250.69.68: seq=0 ttl=128 time=8.959 ms

# Press CTRL + P, then CTRL + Q to detach.

# Launch a second container on the MACvlan network
$ docker run -it --rm --name demo2 --network macvlan1 --ip 10.7.1.201 busybox

# Check MAC and IP details
/ # ip ad 
19: eth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether b6:9c:4e:30:dd:62 brd ff:ff:ff:ff:ff:ff
    inet 10.7.1.201/24 brd 10.7.1.255 scope global eth0

# Test inter-container ping over MACvlan
/ # ping -c 3 10.7.1.200
PING 10.7.1.200 (10.7.1.200): 56 data bytes
64 bytes from 10.7.1.200: seq=0 ttl=128 time=1.049 ms

# Test internet access
/ # ping -c 3 www.google.com
PING www.google.com (142.250.69.68): 56 data bytes
64 bytes from 142.250.69.68: seq=0 ttl=128 time=8.959 ms

# Press CTRL + P, then CTRL + Q to detach.

# Inspect MACvlan network status
$ docker inspect macvlan1
 "ConfigOnly": false,
        "Containers": {
            "2fa4dcce44e4b54129cfd59a2608310964a3aeafa24ed89fa9e08017524a2a90": {
                "Name": "demo2",
                "EndpointID": "139b894b6912f0c58955976de78ac805c67677e909c39a3d3f1200e535bf7f40",
                "MacAddress": "b6:9c:4e:30:dd:62",
                "IPv4Address": "10.7.1.201/24",
                "IPv6Address": ""
            },
            "d15b02111a90accdd465eb4a69d06d11967d5d34fa43d79e07c3ac8661e22d99": {
                "Name": "demo1",
                "EndpointID": "82ae2dbdb988c3b9f585c43e6595357a71791bea27be90dc3edd79c0cd84562e",
                "MacAddress": "ce:03:3e:9b:f9:bf",
                "IPv4Address": "10.7.1.200/24",
                "IPv6Address": ""
            }
        },

# Clean up containers
$ docker rm -f demo1 demo2
```

### Removing a MACvlan Network

```yaml
# Delete MACvlan network
$ docker network rm macvlan1

# Confirm removal
$ docker network ls
```

---

## 4. IPvlan Network

**IPvlan** assigns each container **its own unique IP address (IPv4/IPv6)** while either **sharing the parent interface's physical MAC address (Layer 2 mode)** or **routing packets behind the parent interface (Layer 3 mode)**.

### IPvlan L2 (Layer 2 Mode)

- All egress packets share **the exact same MAC address as the host's parent interface**.
- Each container receives **an IP in the same subnet as the host's parent interface**.
- Useful in environments where **switches prohibit multiple MAC addresses on a single port** (e.g., Wi-Fi networks, VMware security policies, public cloud providers).
- *Limitation:* Host-to-container communication is blocked by default in Linux kernel IPvlan design (can be bypassed by creating an IPvlan sub-interface on the host).

![IPvlan L2 Network Diagram](../images/network5.png?height=500)

#### Example: Creating an IPvlan (Layer 2) Network

```yaml
# Identify parent interface, IP, and default gateway
$ nmcli 
enp0s3: connected to enp0s3
        ethernet, 00:0C:29:08:F5:AB, hw, mtu 1500
        ip4 default
        inet4 10.7.1.232/24
        route4 10.7.1.0/24 metric 100
        route4 default via 10.7.1.2 metric 100

# Create an IPvlan network in Layer 2 mode (shares host MAC)
$ docker network create -d ipvlan --subnet 10.7.1.0/24 --gateway 10.7.1.2 -o parent=enp0s3 ipvlan1

# Verify creation
$ docker network ls

# Run container attached to IPvlan network
$ docker run -it --rm --name demo1 --network ipvlan1 --ip 10.7.1.200 busybox

# Check container MAC address (matches parent host interface MAC)
/ # ip ad 
12: eth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 00:0c:29:08:f5:ab brd ff:ff:ff:ff:ff:ff
    inet 10.7.1.200/24 brd 10.7.1.255 scope global eth0

# Test network connectivity
/ # ping -c 3 www.google.com
PING www.google.com (142.250.69.68): 56 data bytes
64 bytes from 142.250.69.68: seq=0 ttl=128 time=8.959 ms

# Press CTRL + P, then CTRL + Q to detach.

# Inspect IPvlan L2 configuration
$ docker inspect ipvlan1
 "ConfigOnly": false,
        "Containers": {
            "a28a2d74a5917d23b966f026a71ffa344b2cb45a4d094866de3c30a7360ef643": {
                "Name": "demo1",
                "EndpointID": "4a595656aada99509561678baebc648f5fe37c8f2690d94a2a4790b3acfd981c",
                "MacAddress": "",
                "IPv4Address": "10.7.1.200/24",
                "IPv6Address": ""
            },

# Clean up container
$ docker rm -f demo1
```

#### Removing an IPvlan (L2) Network

```yaml
# Remove IPvlan network
$ docker network rm ipvlan1

# Confirm removal
$ docker network ls
```

---

### IPvlan L3 (Layer 3 Mode)

- Containers reside in **one or multiple subnets distinct from the parent interface subnet**.
- **The Docker host acts as an L3 router** for these subnets.
- For external hosts to reach containers directly, **static routes must be added to the upstream router** or NAT rules set on the host.

![IPvlan L3 Network Diagram](../images/network6.png?height=490)

#### Example: Creating an IPvlan (Layer 3) Network

```yaml
# Create an IPvlan Layer 3 network configured with two distinct subnets
$ docker network create -d ipvlan --subnet 192.168.94.0/24 --subnet 192.168.95.0/24 -o parent=enp0s3 -o ipvlan_mode=l3 ipvlan2

# Verify creation
$ docker network ls

# Launch a container in the 1st subnet
$ docker run -it --rm --name demo1 --network ipvlan2 --ip 192.168.94.7 busybox

# Check assigned IP address
/ # ip ad 
30: eth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 00:0c:29:08:f5:ab brd ff:ff:ff:ff:ff:ff
    inet 192.168.94.7/24 brd 192.168.94.255 scope global eth0

# Press CTRL + P, then CTRL + Q to detach.

# Launch a container in the 2nd subnet
$ docker run -it --rm --name demo2 --network ipvlan2 --ip 192.168.95.7 busybox

# Check assigned IP address
/ # ip ad
31: eth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 00:0c:29:08:f5:ab brd ff:ff:ff:ff:ff:ff
    inet 192.168.95.7/24 brd 192.168.95.255 scope global eth0
       valid_lft forever preferred_lft forever

# Note that pinging across subnets fails without explicit routing rules
/ # ping 192.168.94.7 # Will fail by default

/ # ping 8.8.8.8 # Will fail by default without external gateway routes

# Press CTRL + P, then CTRL + Q to detach.

# Inspect IPvlan L3 configuration
$ docker inspect ipvlan2  
"Containers": {
            "2ec42157db2ac5ed396f8d707be0a21ca6dc16939bd06385a0c69d10661348b7": {
                "Name": "demo1",
                "EndpointID": "6c4209cb6251085d3dd4f11c0914a958a6be2a7188bdd64f89b7b60e82beeaae",
                "MacAddress": "",
                "IPv4Address": "192.168.94.7/24",
                "IPv6Address": ""
            },
            "835345e08c6cd0d662b90c1ffc9deb7e0a4fa9d3e8f16c726e5ec105053ee399": {
                "Name": "demo2",
                "EndpointID": "f6804575d065254360044337446cc936092762fa5a47ee288a892c1e3aee350d",
                "MacAddress": "",
                "IPv4Address": "192.168.95.7/24",
                "IPv6Address": ""
            }
```

#### Inter-subnet Routing in IPvlan L3

- Inter-subnet routing does not happen automatically. Containers in `192.168.94.0/24` cannot communicate with `192.168.95.0/24` by default.
- **Static routing tables** must be configured on upstream devices or the host system to route traffic between subnets.
- **External inbound/outbound accessibility** similarly requires routing entries on external network routers.

#### Removing an IPvlan (L3) Network

```yaml
# Stop and remove containers
$ docker rm -f demo1 demo2

# Delete network
$ docker network rm ipvlan2

# Verify deletion
$ docker network ls
```

---

## 5. Overlay Network

An **Overlay** network creates a distributed virtual network spanning across all physical worker nodes in a **Docker Swarm** cluster. Overlay networks enable seamless multi-host container communication between Swarm service tasks and standalone containers.

![Overlay Network Diagram](../images/network-overlay.jpg)

> **Note:** Standalone Docker Engine installations (Docker CE without Swarm mode activated) do not enable multi-host Overlay networking out of the box.

---

## 6. None (null) Network

Under the **None** network driver, containers are completely isolated with no network interfaces attached other than the internal loopback interface (`lo`). They have **no network access to external hosts or other containers**.

![None Network Diagram](../images/network-none.jpg?height=300)

### Example: Running a Container with Disabled Networking

```yaml
# Launch container with network stack disabled
$ docker run -it --rm --name demo1 --network none busybox

# Verify that only the loopback interface exists
/ # ip ad 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
/ # exit
```

---

## Docker Network Drivers Summary

| Feature / Network | Bridge | Host | IPvlan | Macvlan | Overlay | None |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Connectivity** | Connects container to internal host LAN and other containers | Removes network isolation between container and host | Associated directly with Linux Ethernet interface | Assigns dedicated MAC address; appears as physical host | Connects containers across multiple Docker host nodes | Completely isolated container with loopback only |
| **Default Usage** | Default Docker network driver | Only one container can bind a given host port at a time | Provides network isolation and direct physical network access | Clones host interface to create virtual interfaces in container | Available with Docker Swarm / EE enabled | Container cannot communicate with any external network or container |
| **Best Used For** | Most standalone container deployment cases | Specific high-performance management tools running on host | Unique routing capabilities with advanced L2/L3 operating modes | Connecting containers directly to physical VLANs/LANs | Multi-host cluster networking using VXLAN encapsulation | Complete container isolation (e.g., batch jobs, security sandboxes) |

---

## Resources

- Official Docker Networking Documentation: https://docs.docker.com/network/


