---
layout: post
title: Container Linking
date: 2015-09-26
comments: true
archive: false
---

When inter container communication (icc) is enabled with Docker, containers on the same host can be linked together to work as unit. 

Let's say we want to build app which requires a Database and Web server, we can :-

- put both of them in one container
- put them a different container but on same system
- put them on different systems in different systems, in the same cluster

In this recipe, we will look how we can link diffent container on the same system

<script type="text/javascript" src="https://asciinema.org/a/26945.js" id="asciicast-26945" async  data-theme="solarized-dark"></script>

```
$ vagrant ssh labvm-1
$ sudo -s
$ docker run -d  -e MYSQL_ROOT_PASSWORD=my-secret-pw  --name=db mysql
$ docker inspect --format='{{.NetworkSettings.IPAddress}}' db
$ docker run -itd --name web --link db:database centos
$ docker ps
$ docker inspect -f "{{ .HostConfig.Links }}" web
$ docker exec -it web bash
$ ip a
$ cat /etc/hosts
$ env
$ ping $DATABASE_PORT_3306_TCP_ADDR
```
