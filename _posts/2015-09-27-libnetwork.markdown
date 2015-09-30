---
layout: post
title: Docker Libnetwork
date: 2015-09-27
comments: true
archive: false
---
## Docker Networking design as of Docker v1.6

Prior to libnetwork, Docker Networking was handled in both Docker Engine and libcontainer.
Docker Engine makes use of the Bridge Driver to provide single-host networking solution with the help of linux bridge and IPTables.
Docker Engine provides simple configurations such as `--link`, `--expose`,... to enable container connectivity within the same host by abstracting away networking configuration completely from the Containers.
For external connectivity, it relied upon NAT & Port-mapping 

Docker Engine was responsible for providing the configuration for the container's networking stack.

Libcontainer would then use this information to create the necessary networking devices and move them in to a network namespace.
This namespace would then be used when the container is started.
https://github.com/docker/libnetwork/blob/master/docs/legacy.md


libnetwork project will follow Docker and Linux philosophy of developing small, highly modular and composable tools that works well independently. Libnetwork aims to satisfy that composable need for Networking in Containers.

Replace the networking subsystem of Docker Engine, with libnetwork
https://github.com/docker/libnetwork/blob/master/ROADMAP.md#project-planning

<script type="text/javascript" src="https://asciinema.org/a/26992.js" id="asciicast-26992" async></script>
~~~
[root@lab-vm-1 ~] iptables -F; systemctl stop docker
[root@lab-vm-1 ~] ./consul agent -server -bootstrap -data-dir /tmp/consul -bind=192.168.100.23> /dev/null 2>&1 &

[root@lab-vm-2 ~] ./consul agent -data-dir /tmp/consul -bind 192.168.100.24 > /dev/null 2>&1 &
[root@lab-vm-2 ~] ./consul join 192.168.100.23

[root@lab-vm-1 ~] ./docker-latest -d --kv-store=consul:localhost:8500 --label=com.docker.network.driver.overlay.bind_interface=eth1 > /dev/null 2>&1 &

[root@lab-vm-2 ~]./docker-latest -d --kv-store=consul:localhost:8500 --label=com.docker.network.driver.overlay.bind_interface=eth1 --label=com.docker.network.driver.overlay.neighbor_ip=192.168.100.23 > /dev/null 2>&1 &

[root@lab-vm-1 ~] ./docker-latest run -itd --publish-service=svc1.dev.overlay --name container_node1 docker.io/centos 

[root@lab-vm-2 ~]./docker-latest run -itd --publish-service=svc2.dev.overlay --name container_node2 docker.io/centos

./docker-latest network ls
./docker-latest service ls

[root@lab-vm-2 ~] ./docker-latest exec -it container_node2 bash
[root@31171f3e1da0 /]# cat /etc/hosts
..

ping svc1

[root@lab-vm-1 ~]# tcpdump -i eth1 
~~~

<script type="text/javascript" src="https://asciinema.org/a/26993.js" id="asciicast-26993" async></script>

~~~
[root@lab-vm-1 ~] ./docker-latest network ls

[root@lab-vm-1 ~]  cd /var/run/docker/netns/

[root@lab-vm-1 ~] nsenter --net=1-3aa4e1aa64 ip link show

[root@lab-vm-1 ~] nsenter --net=1-3aa4e1aa64 ip neigh show

[root@lab-vm-2 ~] ./docker-latest ls

[root@lab-vm-2 ~]  ./docker-latest exec -it container_node2 bash
[root@31171f3e1da0 /]# ip a 

[root@lab-vm-1 ~] nsenter --net=1-3aa4e1aa64  bridge fdb show

~~~

