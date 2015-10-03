---
layout: post
date: 2015-09-23
comments: true
archive: false
title: Volume Plugins
---

With Docker Volume Plugins we can enable container to access external storage natively. We would see an example of [Gluster Volume plugin](https://github.com/calavera/docker-volume-glusterfs).

<script type="text/javascript" src="https://asciinema.org/a/27058.js" id="asciicast-27058" async></script>

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

