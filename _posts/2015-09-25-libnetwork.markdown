---
layout: post
date: 2015-09-25
title: Libnetwork
comments: true
archive: false
---

As we scale out with containers, we would need containers from one host to talk to containers from other host. There are few solutions, which are tying to  solve this like [Flannel](https://github.com/coreos/flannel), [Weave](https://github.com/weaveworks/weave), [Calico](http://www.projectcalico.org/) etc.

With container commnunication we would also need see how containers discover each other and for that we need some kind of container service discovery mmechanism, which can be achived by some form shared key-value store. For example Flannel uses [`etcd`](https://github.com/coreos/etcd) as `kv` store.

Having lots different options are good but they does not provide very good user experience. With [`libnetwork`](https://github.com/docker/libnetwork), Docker is aiming to provide standard interface to connect containers and satisfy composible needs. 
 
<iframe src="//www.slideshare.net/slideshow/embed_code/key/b4sWkoqbwubDMR?startSlide=11" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/Docker/networking-breakout-v03" title="Docker Online Meetup #22: Docker Networking" target="_blank">Docker Online Meetup #22: Docker Networking</a> </strong> from <strong><a href="//www.slideshare.net/Docker" target="_blank">Docker, Inc.</a></strong> </div>

## The Container Network Model

Libnetwork implements Container Network Model (CNM) which formalizes the steps required to provide networking for containers while providing an abstraction that can be used to support multiple network drivers. The CNM is built on 3 main components.

**Sandbox**

A Sandbox contains the configuration of a container's network stack.
This includes management of the container's interfaces, routing table and DNS settings.
An implementation of a Sandbox could be a Linux Network Namespace, a FreeBSD Jail or other similar concept.
A Sandbox may contain *many* endpoints from *multiple* networks.

**Endpoint**

An Endpoint joins a Sandbox to a Network.
An implementation of an Endpoint could be a `veth` pair, an Open vSwitch internal port or similar.
An Endpoint can belong to *only one* network but may only belong to *one* Sandbox.

**Network**

A Network is a group of Endpoints that are able to communicate with each-other directly.
An implementation of a Network could be a Linux bridge, a VLAN, etc.
Networks consist of *many* endpoints.


## Libnetwork Drivers

### Null

The null driver is a `noop` implementation of the driver API, used only in cases where no networking is desired. This is to provide backward compatibility to the Docker's `--net=none` option.

### Bridge

The `bridge` driver provides a Linux-specific bridging implementation based on the Linux Bridge.

### Overlay

The `overlay` driver implements networking that can span multiple hosts using overlay network encapsulations such as VXLAN. 

### Remote

The `remote` package does not provide a driver, but provides a means of supporting drivers over a remote transport. This can be used to write third party plugin for Docker.

## Demo
For this demo we would use `overlay` driver and configure overlay network between two systems. We would use `consul` as `kv` pair here. Any other `kv` pair like `etcd` can be used. Once configured we would access container on one system from contaier on other system. 

We would see that container can reach out to each other with IP address and service name. We would also collect the `tcpdump` output on one of container host machine and see how VXLAN packet look like.

<script type="text/javascript" src="https://asciinema.org/a/26992.js" id="asciicast-26992" async  data-theme="solarized-dark"></script>
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

Now lets try to dig a bit and look at how things work in background. 

<script type="text/javascript" src="https://asciinema.org/a/26993.js" id="asciicast-26993" async  data-theme="solarized-dark"></script>

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

