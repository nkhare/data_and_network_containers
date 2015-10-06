---
layout: post
date: 2015-09-23
comments: true
archive: false
title: Volume Plugins
---

With Docker Volume Plugins we can enable container to access external storage natively. Currently [plugins](https://docs.docker.com/extend/plugins/) are available only on experimental release. There are few existing volume plug-ins like [Blockbridge Plugin](https://github.com/blockbridge/blockbridge-docker-volume), [Flocker](https://clusterhq.com/docker-plugin/), [GlusterFS](https://github.com/calavera/docker-volume-glusterfs). 

In this demo we would use GlusterFS plugin with Docker experimental release. We would configure Gluster volume in one VM and using the Docker plugin, we would attach that volume to a container running to different machine.  

<script type="text/javascript" src="https://asciinema.org/a/27058.js" id="asciicast-27058" async  data-theme="solarized-dark"></script>

~~~
vagrant ssh labvm-1
sudo -s
cd
iptables -F
systemctl start glusterd
mkdir -p /mnt/brick
gluster volume create dockervol 192.168.100.23:/mnt/brick force
gluster volume start dockervol 
~~~

~~~
vagrant ssh labvm-2
sudo -s
cd
iptables -F
mkdir /tmp/vol
mount -t glusterfs 192.168.100.23:/dockervol /tmp/vol
export GOPATH=/root/code
/root/code/bin/docker-volume-glusterfs -servers 192.168.100.23 &
systemctl stop docker
./docker-latest daemon  > /dev/null 2>&1  &
./docker-latest  run -it  --volume-driver glusterfs --volume dockervol:/data centos
touch  /data/file
~~~

