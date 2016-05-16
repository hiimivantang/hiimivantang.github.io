---
layout: post
title:  "installing docker offline"
date:   2016-05-16 14:41
---

This is a guide on how to install Docker on a Ubuntu machine (64bit) without internet access.


### Getting ready
Using another machine with internet access, download the following:
* Docker Engine Binary
* cgroup-lite package (if necessary)


How to Download Docker Engine Binary:

Use the following URLs for downloading Docker engine binary

```bash
#32bit
https://get.docker.com/builds/Linux/i386/docker-latest.tgz

#Intel or AMD 64bit 
https://get.docker.com/builds/Linux/x86_64/docker-latest.tgz

```

Make sure you have cgroups-list installed. if not, You can find the cgroup-lite package file for trusty [here][1]



### Transfer Docker Engine binary and cgroup-lite package

Copy the Docker Engine binary and cgroup-lite package to your machine without internet access

extract the contents of the Docker archive:
```bash
tar -xvzf docker-latest.tgz

```

Install the binaries extracted to /usr/bin:
```bash
mv docker/* /usr/bin/
```

If necessary, install the cgroup-lite package:
```bash
dpkg -i cgroup-lite_*.deb
```

### Setting up upstart script for Docker

Nobody wants to manually start a service whenever your machine boots. This is why [upstart][2] exists. Download bo




[1]: http://packages.ubuntu.com/trusty/cgroup-lite
[2]: http://upstart.ubuntu.com/
