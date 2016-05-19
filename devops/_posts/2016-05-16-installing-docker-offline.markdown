---
layout: post
title:  "Installing Docker engine offline"
date:   2016-05-16 14:41
tags: docker ubuntu
---

This is a guide on how to install Docker on a Ubuntu machine (64bit) without internet access.
<br>
<br>

### Getting ready
Using another machine with internet access, download the following:

* Docker Engine Binary
* cgroup-lite package (if necessary)

Use the following URLs for downloading Docker engine binary:

```bash
#32bit
https://get.docker.com/builds/Linux/i386/docker-latest.tgz

#Intel or AMD 64bit 
https://get.docker.com/builds/Linux/x86_64/docker-latest.tgz

```

Make sure you have cgroups-list installed. if not, You can find the cgroup-lite package file for trusty [here][1].
<br>
<br>

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
<br>
<br>

### Setting up upstart script for Docker

Nobody wants to manually start a service whenever your machine boots. This is why [upstart][2] exists. 

Download the upstart script from the [official Docker github repo][3] and save the file as /etc/init/upstart-docker.

This was the file I downloaded at the time of writing:

```bash
description "Docker daemon"

start on (filesystem and net-device-up IFACE!=lo)
stop on runlevel [!2345]
limit nofile 524288 1048576
limit nproc 524288 1048576

respawn

kill timeout 20

pre-start script
    # see also https://github.com/tianon/cgroupfs-mount/blob/master/cgroupfs-mount
    if grep -v '^#' /etc/fstab | grep -q cgroup \
        || [ ! -e /proc/cgroups ] \
        || [ ! -d /sys/fs/cgroup ]; then
        exit 0
    fi
    if ! mountpoint -q /sys/fs/cgroup; then
        mount -t tmpfs -o uid=0,gid=0,mode=0755 cgroup /sys/fs/cgroup
    fi
    (
        cd /sys/fs/cgroup
        for sys in $(awk '!/^#/ { if ($4 == 1) print $1 }' /proc/cgroups); do
            mkdir -p $sys
            if ! mountpoint -q $sys; then
                if ! mount -n -t cgroup -o $sys cgroup $sys; then
                    rmdir $sys || true
                fi
            fi
        done
    )
end script

script
    # modify these in /etc/default/$UPSTART_JOB (/etc/default/docker)
    DOCKERD=/usr/bin/dockerd
    DOCKER_OPTS=
    if [ -f /etc/default/$UPSTART_JOB ]; then
        . /etc/default/$UPSTART_JOB
    fi
    exec "$DOCKERD" $DOCKER_OPTS --raw-logs
end script

# Don't emit "started" event until docker.sock is ready.
# See https://github.com/docker/docker/issues/6647
post-start script
    DOCKER_OPTS=
    DOCKER_SOCKET=
    if [ -f /etc/default/$UPSTART_JOB ]; then
        . /etc/default/$UPSTART_JOB
    fi

    if ! printf "%s" "$DOCKER_OPTS" | grep -qE -e '-H|--host'; then
        DOCKER_SOCKET=/var/run/docker.sock
    else
        DOCKER_SOCKET=$(printf "%s" "$DOCKER_OPTS" | grep -oP -e '(-H|--host)\W*unix://\K(\S+)')
    fi

    if [ -n "$DOCKER_SOCKET" ]; then
        while ! [ -e "$DOCKER_SOCKET" ]; do
            initctl status $UPSTART_JOB | grep -qE "(stop|respawn)/" && exit 1
            echo "Waiting for $DOCKER_SOCKET"
            sleep 0.1
        done
        echo "$DOCKER_SOCKET is up"
    fi
end script
```

I've changed the following:

```bash
...

    DOCKERD=/usr/bin/dockerd
    DOCKER_OPTS=
    if [ -f /etc/default/$UPSTART_JOB ]; then
        . /etc/default/$UPSTART_JOB
    fi
    exec "$DOCKERD" $DOCKER_OPTS --raw-logs

...
```

to:

```bash
...

    DOCKERD=/usr/bin/docker
    DOCKER_OPTS=
    if [ -f /etc/default/$UPSTART_JOB ]; then
        . /etc/default/$UPSTART_JOB
    fi
    exec "$DOCKERD" $DOCKER_OPTS daemon --raw-logs

...
```

The dockerd executable is probably not released in the latest stable version of Docker I've downloaded and installed. If you see dockerd in your /usr/bin/, you don't have to change the upstart script.


Next, register your new upstart script by running:

```bash
sudo initctl reload-configuration
```

<br>
<br>

### How to use Docker without sudo

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

Log out and log back in

<br>
<br>

### Enjoy Docker on your machine without internet access

```bash
sudo service upstart-docker start
```



### Bonus: How to install docker-compose


```bash
#log in as root
sudo -i
# download docker-compose binary and install on /usr/local/bin or anywhere in the PATH environment variable
curl -L https://github.com/docker/compose/releases/download/1.7.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

[1]: http://packages.ubuntu.com/trusty/cgroup-lite
[2]: http://upstart.ubuntu.com/
[3]: https://raw.githubusercontent.com/docker/docker/master/contrib/init/upstart/docker.conf
