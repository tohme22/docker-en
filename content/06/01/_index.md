---
title: "Demo"
description: "docker"
draft: true
weight: 2
---
### Commands used

#### Bridge (by default)
```yaml
$ docker info  

$ docker network ls
$ docker network inspect bridge
$ nmcli

$ docker run -itd --rm --name demo1 busybox
$ docker run -itd --rm --name demo2 busybox
$ docker run -itd --rm --name demo3 busybox

$ nmcli
$ bridge link
$ docker inspect bridge
$ cat /etc/resolv.conf

$ docker exec -it demo1 sh
/ # ip ad
/ # ping 172.17.0.3
/ # ping www.google.com
/ # ip route
/ # exit

$ docker stop demo1 demo2 demo3
```

#### Bridge (predefinied)
```yaml
$ docker network create --driver bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 myapp-net
$ docker network ls
$ docker network inspect myapp-net
$ nmcli

$ docker run -it --name app1 --network myapp-net alpine
/ # ip ad

From another terminal:
$ docker network inspect myapp-net  
$ docker inspect app1
$ docker run -it --name app2 --network myapp-net alpine
/ # ip ad
/ # ping 192.168.1.2
/ # ping www.google.com
/ # ping app1
/ # exit

Go the prvious terminal, to exit from app1:
/ # exit

$ docker network ls
$ docker network rm myapp-net
$ docker network ls

$ docker ps -a 
docker start app1
Error response from daemon: network 2d75cb4e10626e0b791110d7b71 not found
Error: failed to start containers: app1

$ docker network disconnect myapp-net app1
$ docker network create --driver bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 myapp-net
$ docker network connect myapp-net app1
$ docker start -i app1
/ # ip ad
/ # exit

$ docker rm app1 app2 
$ docker network prune  (Remove all unused networks)
$ docker network ls
```

#### Host
```yaml
$ docker network ls
$ docker run --rm -d --name web1 --network host nginx
$ docker network inspect host
$ nmcli --> to get the host ip adress

$ docker network rm host
Error response from daemon: host is a pre-defined network and cannot be removed
```

#### MACVlan
```yaml
$ nmcli --> To get the ip adress of the host network interface (ex: ens160)
$ docker network create -d macvlan --subnet 192.168.65.0/24 --gateway 192.168.65.2 -o parent=ens160 macvlan1
$ docker network ls
$ docker run -it --rm --name demo1 --network macvlan1 --ip 192.168.65.200 busybox
/ # ip ad --> Check the MAC adress
/ # ping www.google.com

From another terminal:
$ docker run -it --rm --name demo2 --network macvlan1 --ip 192.168.65.201 busybox
/ # ip ad –> Compare with demo1 MAC adress
/ # ping 192.168.65.200
/ # ping 192.168.65.2
/ # exit

$ docker stop demo1
$ docker network rm macvlan1
$ docker network ls
```

#### IPVlan (Layer 2)
```yaml
$ docker network create -d ipvlan --subnet 192.168.65.0/24 --gateway 192.168.65.2 -o parent=ens160 ipvlan1
$ docker network ls
$ docker run -it --rm --name demo1 --network ipvlan1 --ip 192.168.65.200 busybox
/ # ip ad --> Compare it with the host MAC adress _(It is similar)_
/ # ping www.google.com
/ # exit
$ docker network rm ipvlan1
$ docker network ls
```

#### IPVlan (Layer 3)
```yaml
$ docker network create -d ipvlan --subnet 192.168.94.0/24 -o parent=ens160 -o ipvlan_mod=l3 --subnet 192.168.95.0/24 ipvlan2
$ docker network ls
$ docker run -it --rm --name demo1 --network ipvlan2 --ip 192.168.94.7 busybox
/ # ip ad 

From another terminal (To create a container in another subnet):
$ docker run -it --rm --name demo2 --network ipvlan2 --ip 192.168.95.7 busybox
/ # ip ad
/ # ping 192.168.94.7 (will not work)
/ # ping 8.8.8.8 (will not work)
We need to create static route to be able to connect this new subnets.

From another 3rd terminal:
$ docker inspect ipvlan2   
$ docker stop demo1 demo2
$ docker network rm ipvlan2 
$ docker network ls
```

#### NONE 
```yaml
$ docker run -it --rm --name demo3 --network none busybox
/ # ip ad --> No IP adress
/ # exit
```








