---
layout: post
date: 2015-09-24
comments: true
archive: false
title: Storage Backend
---

What happens when container starts

Whats not happen
- did not copy full image
- did not modify orignal image
- did not effect other containers

## Copy On Write (CoW)
- Implicit Sharing
 * looks like copy but its just a reference to original resource
 * Do not create the local copy until modification is required. 

### Where CoW  is used
- Keep track of changes b/w the image and our container
- Process Creation *fork()*
- Memory Snapshot *redis*
- Mapped Memory *mmap()*
- Disk Snapshot *btrfs*
- VM Provisioning *Vagrant*
- Containers 



### Linux
```
# pwd
/work/docker/docker/daemon/graphdriver
# grep -ir -A9 "priority = \[\]string" driver_linux.go 
	priority = []string{
		"aufs",
		"btrfs",
		"zfs",
		"devicemapper",
		"overlay",
		"vfs",
	}
```

### Windows
```
# grep -ir -A4 "priority = \[\]string" driver_windows.go 
	priority = []string{
		"windowsfilter",
		"windowsdiff",
		"vfs",
	}
```

### Freebsd
```
# grep -ir -A2 "priority = \[\]string" driver_freebsd.go 
	priority = []string{
		"zfs",
	}
```

### Unsupported
```
# grep -ir -A2 "priority = \[\]string" driver_unsupported.go 
	priority = []string{
		"unsupported",
	}
```


## [AUFS](https://github.com/docker/docker/tree/master/daemon/graphdriver/aufs)

<a href="AUFS"><img src="http://cdn-ak.f.st-hatena.com/images/fotolife/d/dayflower/20080714/20080714131209.png" align="center" height="200" width="300" ></a>

- Not in main line kernel 
- CoW  works at file level
- Before writing, file has to be copied at upper most layer

As it work at file level, AUFS can take benefit *Linux page cache*. So with Docker, starting container from same image is very fast.

**Performance Issuses**

- Copying large file from read-only layer  
- As the number layers increases, penalty for looking up a file increases


## [Device Mapper](https://github.com/docker/docker/blob/master/daemon/graphdriver/devmapper)
Device mapper creates logical devices on top of pysical block device and proides addtional feature likes 

- RAID (dm-raid)
- Multipath (dm-multipath)
- Encyption (dm-crypt)
- Delay (dm-delay)
- Thin Provision (dm-thin)
  * Used to creare snapshots using CoW
  * Works at Block Level

**As thinp CoW  works at the block level, it can not take benefit of *page cache* as we saw with AUFS**

<iframe src="//www.slideshare.net/slideshow/embed_code/key/ttfJKXGN6vtHLE?startSlide=16" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/enakai/docker-technology-v18e" title="Inside Docker for Fedora20/RHEL7" target="_blank">Inside Docker for Fedora20/RHEL7</a> </strong> from <strong><a href="//www.slideshare.net/enakai" target="_blank">Etsuji Nakai</a></strong> </div>

<iframe src="//www.slideshare.net/slideshow/embed_code/key/ttfJKXGN6vtHLE?startSlide=19" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/enakai/docker-technology-v18e" title="Inside Docker for Fedora20/RHEL7" target="_blank">Inside Docker for Fedora20/RHEL7</a> </strong> from <strong><a href="//www.slideshare.net/enakai" target="_blank">Etsuji Nakai</a></strong> </div>

By default Docker uses Device Mapper and configure it on top of loopback devices, which are not performant and not recommened for prodcution use. To get better performance we should use put Thin Provisioned volumes on real block devices. 

Configutation option for [Device Mapper](https://github.com/docker/docker/blob/master/daemon/graphdriver/devmapper/README.md):-

 *  `Pool Name` name of the devicemapper pool for this driver.
 *  `Pool Blocksize` tells the blocksize the thin pool was initialized with. This only changes on creation.
 *  `Base Device Size` tells the maximum size of a container and image
 *  `Data file` blockdevice file used for the devicemapper data
 *  `Metadata file` blockdevice file used for the devicemapper metadata
 *  `Data Space Used` tells how much of `Data file` is currently used
 *  `Data Space Total` tells max size the `Data file`
 *  `Data Space Available` tells how much free space there is in the `Data file`. If you are using a loop device this will report the actual space available to the loop device on the underlying filesystem.
 *  `Metadata Space Used` tells how much of `Metadata file` is currently used
 *  `Metadata Space Total` tells max size the `Metadata file`
 *  `Metadata Space Available` tells how much free space there is in the `Metadata file`. If you are using a loop device this will report the actual space available to the loop device on the underlying filesystem.
 *  `Udev Sync Supported` tells whether devicemapper is able to sync with Udev. Should be `true`.
 *  `Data loop file` file attached to `Data file`, if loopback device is used
 *  `Metadata loop file` file attached to `Metadata file`, if loopback device is used
 *  `Library Version` from the libdevmapper used


## [Btrfs](https://github.com/docker/docker/tree/master/daemon/graphdriver/btrfs)
- CoW  at filesystem level, instead of file or block
- Snapshot is created from a subvolume 
- btrfs should created  on /var/lib/docker to use with Docker 

## [Overlay](https://github.com/docker/docker/tree/master/daemon/graphdriver/overlay)
<a href="AUFS"><img src="http://www.blaess.fr/christophe/wp-content/uploads/2014/12/overlayfs.png" align="center" height="200" width="300" ></a>

- Written to do filesystem virualization
- In main line kernel
- Works at file	level
- Shares the page cache, when files are open in read only in upper layer.

Lower Layer
Upper Layer
Workdir
Overlay layer (switch)

mount 

- User interface is pathwalk 
- Works by path co-incidence
- Overlay layer switches  between layer
- Lower layer copied up on change

## [ZFS](https://github.com/docker/docker/tree/master/daemon/graphdriver/zfs)

## [VFS](https://github.com/docker/docker/tree/master/daemon/graphdriver/vfs)

## Links
- https://jpetazzo.github.io/assets/2015-07-01-deep-dive-into-docker-storage-drivers.html
- http://developerblog.redhat.com/2014/09/30/overview-storage-scalability-docker/
- http://www.projectatomic.io/docs/docker-storage-recommendation/
- http://www.projectatomic.io/blog/2015/06/notes-on-fedora-centos-and-docker-storage-drivers/
