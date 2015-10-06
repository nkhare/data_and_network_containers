---
layout: post
date: 2015-09-27
comments: true
archive: false
title: Docker Networking
---

In this section we are going to look at state of Networking with current Docker release 1.8. More details can be found at [Docker Documentation](https://docs.docker.com/articles/networking/) 

## What happens by default

As soon as you start the Docker daemon, it would create a Linux Brigde (virtual Ethernet bridge) called `docker0`, that automatically forwards packets between any other network interfaces that are attached to it. As a new container would start,  Docker would create a peer of network interfaces and attach one end to the container and other to the bridge `docker0`. 

<script type="text/javascript" src="https://asciinema.org/a/26765.js" id="asciicast-26765" data-size="small" async  data-theme="solarized-dark"></script>

Docker has some global network configuration options which effects all the containers and some per container configurtion options. Global options are passed to the Docker daemon. Some of the global configuration options are :-

- `-b BRIDGE or --bridge=BRIDGE` to use custom bridge
- `--default-gateway=IP_ADDRESS` to set the default route for containers
- `--icc=true|false` to enable/disable container communication

Visit [Docker Documentation](https://docs.docker.com/articles/networking/) for more details.

And the per conatainer options are:-

- `-h HOSTNAME or --hostname=HOSTNAME` to set the hostname for container
- `--link=CONTAINER_NAME_or_ID:ALIAS`  to link the container with other contaiiners
- `--net=bridge|none|container:NAME_or_ID|host` to connect the container with differnt namespaces
- `-p SPEC or --publish=SPEC` 	to bind the container with specific port of host 
- `-P or --publish-all=true|false` to bind all the exposed port from a container to host ports (from `ephemeral port` range)  

## `--net` Networking options

### --net=bridge

<iframe src="//www.slideshare.net/slideshow/embed_code/key/wiDeE3NzpScr4y?startSlide=8" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/SreenivasMakam/docker-networking-current-status-and-goals-of-experimental-networking/8" title="Docker Networking - Current Status and goals of Experimental Networking" target="_blank">Docker Networking - Current Status and goals of Experimental Networking</a> </strong> from <strong><a href="//www.slideshare.net/SreenivasMakam" target="_blank">Sreenivas Makam</a></strong> </div>


~~~
$ vagrant ssh labvm-1
$ sudo -s
$ ip a
$ docker run -itd centos bash
$ ip a
$ docker ps
$ docker exec -it <ID> ip a
~~~

<script type="text/javascript" src="https://asciinema.org/a/26873.js" id="asciicast-26873" async  data-theme="solarized-dark"></script>
~~~
$ vagrant ssh labvm-1
$ sudo -s
$ docker run -itd --name web centos bash
$ docker run -itd --name db mysql bash
$ brctl-show docker0
$ iptabales -t nat -n -L POSTROUTING
$ docker ps
$ docker exec -it web bash
$ tracepath redhat.com
~~~


### --net=host
 
With `--net=host` option, Docker would not create `network` namespace for the container and would share the namespace of the host.

<script type="text/javascript" src="https://asciinema.org/a/26811.js" id="asciicast-26811" async  data-theme="solarized-dark"></script>

~~~
$ vagrant ssh labvm-1
$ sudo -s
$ ip a
$ ip adocker run -it  centos bash
$ docker run -it  --net=host centos bash
$ ip a
~~~

### --net=container:NAME_or_ID
With above Docker would also not create new `network` namespace of container but it would share it with other container. In [Kubernetes](http://kubernetes.io/), a pod can consist of multiple container and it uses `--net=container:NAME_or_ID` trick to share same namespace among them .

<script type="text/javascript" src="https://asciinema.org/a/26813.js" id="asciicast-26813" async  data-theme="solarized-dark"></script>

~~~
$ vagrant ssh labvm-1
$ sudo -s
$ docker run -itd --name database mysql bash
$ docker exec -it database ip a
$ docker run -it --net=container:database centos  bash
$ ip a
~~~


### --net=none
With `--net=none`, Docker would not create any namespace for the container. 

<script type="text/javascript" src="https://asciinema.org/a/26814.js" id="asciicast-26814" async  data-theme="solarized-dark"></script>

~~~
$ vagrant ssh labvm-1
$ sudo -s
$ ip adocker run -it --net=none  centos bash
$ ip a
~~~


## Accessing the container from outside world 

In case of --net=host, container can be accessed through host IP. In other cases,  if you would like to access the containers from outside, then we can a  map host port with the container port and forward the traffic from host to container using `iptables`. 

<script type="text/javascript" src="https://asciinema.org/a/26922.js" id="asciicast-26922" async  data-theme="solarized-dark"></script>

~~~
$ vagrant ssh labvm-1
$ sudo -s
$ docker run -d  -e MYSQL_ROOT_PASSWORD=my-secret-pw  --name=db mysql
$ docker ps
$ docker inspect --format='{{.NetworkSettings.IPAddress}}' db
$ telnet <IP> 3306
$ telnet localhost 3306
$ docker run -d  -e MYSQL_ROOT_PASSWORD=my-secret-pw  --name=db1 -P 3306 mysql
$ docker ps 
$ tenet localhost <PORT>
$ docker run -d  -e MYSQL_ROOT_PASSWORD=my-secret-pw  --name=db2 -p 3306:3306 mysql
$ docker ps
$ telnet localhost 3306
$ iptables -t nat -n -L  DOCKER
~~~


Other examples

```
$ docker run -i -d -p 192.168.1.10::22 --name f20 fedora /bin/bash
```

We can  bind multiple ports on container to hosts ports like following:-

```
$  docker run -d -i -p 5000:22 -p 8080:80 --name f20 fedora /bin/bash
```
