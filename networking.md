---
layout: recipe
title: docker networking
---

* TOC
{:toc}

## What happens by default
<script type="text/javascript" src="https://asciinema.org/a/26765.js" id="asciicast-26765" data-size="small" async></script>


## Networking options

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

