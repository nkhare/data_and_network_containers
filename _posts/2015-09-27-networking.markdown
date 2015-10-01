---
layout: post
date: 2015-09-27
comments: true
archive: false
title: docker networking
---

## What happens by default
<script type="text/javascript" src="https://asciinema.org/a/26765.js" id="asciicast-26765" data-size="small" async></script>


## Networking options

<iframe src="//www.slideshare.net/slideshow/embed_code/key/wiDeE3NzpScr4y?startSlide=8" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/SreenivasMakam/docker-networking-current-status-and-goals-of-experimental-networking/8" title="Docker Networking - Current Status and goals of Experimental Networking" target="_blank">Docker Networking - Current Status and goals of Experimental Networking</a> </strong> from <strong><a href="//www.slideshare.net/SreenivasMakam" target="_blank">Sreenivas Makam</a></strong> </div>

### --net=bridge
~~~
$ vagrant ssh labvm-1
$ sudo -s
$ ip a
$ docker run -itd centos bash
$ ip a
$ docker ps
$ docker exec -it <ID> ip a
~~~

<script type="text/javascript" src="https://asciinema.org/a/26873.js" id="asciicast-26873" async></script>
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
<script type="text/javascript" src="https://asciinema.org/a/26811.js" id="asciicast-26811" async></script>

~~~
$ vagrant ssh labvm-1
$ sudo -s
$ ip a
$ ip adocker run -it  centos bash
$ docker run -it  --net=host centos bash
$ ip a
~~~

### --net=container:NAME_or_ID
<script type="text/javascript" src="https://asciinema.org/a/26813.js" id="asciicast-26813" async></script>

~~~
$ vagrant ssh labvm-1
$ sudo -s
$ docker run -itd --name database mysql bash
$ docker exec -it database ip a
$ docker run -it --net=container:database centos  bash
$ ip a
~~~


### --net=none
<script type="text/javascript" src="https://asciinema.org/a/26814.js" id="asciicast-26814" async></script>

~~~
$ vagrant ssh labvm-1
$ sudo -s
$ ip adocker run -it --net=none  centos bash
$ ip a
~~~


## Accessting the container from outside world 
### In case of --net=host, container can can be accessed through host IP

<script type="text/javascript" src="https://asciinema.org/a/26922.js" id="asciicast-26922" async></script>

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
